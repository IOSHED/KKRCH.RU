<a id="post-shorts-create"></a>

### <span style="background:#42A5F5;padding:5px">POST</span> `/shorts/{scope_id:int}`

|                      | Описание                                                                                                                                     |
|----------------------|----------------------------------------------------------------------------------------------------------------------------------------------|
| **Назначение**       | Создаёт укороченную ссылку                                                                                                                   |
| **Auth**             | Bearer или X-Api-Key                                                                                                                         |
| **Логика**           | 1. Валидирует поля: `targets`, `short_name`, `subdomain`, `tags`, `set_password`, UTM/CPC                                                    |
|                      | &nbsp;&nbsp;&nbsp;- `set_password` и `is_captcha=true` взаимоисключающие → 400 `validation_error` (оба фильтра пройти нельзя)                |
|                      | 2. `Σ targets[].weight = 100`; каждый `url` на любой домен из `short.base_domains` (или его поддомен) → 400 `recursive_redirect_error`       |
|                      | 3. Ссылка создаётся сразу; Yandex Safe Browsing — async после commit (`url_safety_needs_recheck`); при Unsafe → `inactive_reason=url_unsafe` |
|                      | 4. Если `set_password` задан — хеширует пароль через Argon2 (параметры из `short.argon2` конфига)                                            |
|                      | 5. Если `short_name` содержит `{random_suffix=N}` — разрешает шаблон через `RandomSuffixCalculatorService`:                                  |
|                      | &nbsp;&nbsp;&nbsp;- N ≤ порога: sequential base58-счётчик (таблица `short_id_blocks`)                                                        |
|                      | &nbsp;&nbsp;&nbsp;- N > порога (+ есть subdomain): random + `short_name_reservations` + retry                                                |
|                      | 6. Сохраняет short + `short_targets` в Postgres                                                                                              |
|                      | 7. Прогревает Redis resolve-кеш (`shorts:resolve:{ns}:{name}`) с TTL `short.resolve_cache_ttl`                                           |
|                      | 8. После commit инвалидирует кеш availability и обновляет bloom-фильтр                                                                   |
| **Параметры**        | `scope_id:int` - индификатор scope                                                                                                           |
| **Инвалидация кеша** | `INCR shorts:scope:{scope}:v`, очистка `shorts:availability:{subdomain}:{short_name}`, bloom add; **прогрев** resolve-кеша                 |

Контракт `targets` / CPC / UTM — в [module_short.md](../module_short.md#targets--cpc-несколько-destination-url).

---

| Ответ                        | Код | Описание                                                                                |
|------------------------------|-----|-----------------------------------------------------------------------------------------|
|                              | 201 | Создана короткая ссылка                                                                 |
| long_url_validation_error    | 400 | Некорректный `targets[].url` (не http/https)                                            |
| targets_validation_error     | 400 | Пустой targets / `len > max_targets` / `weight=0` / сумма weight ≠ 100                  |
| cpc_budget_validation_error  | 400 | `target.budget` задан без `target.cpc` (`cpc` без `budget` — ок)                        |
| utm_validation_error         | 400 | `utm.required=true` без непустых source/medium/campaign                                 |
| max_clicks_validation_error  | 400 | Глобальный `max_clicks` < суммы заданных `targets[].max_clicks`                         |
| budget_validation_error      | 400 | Глобальный `budget` < суммы заданных `targets[].budget`                                 |
| macro_validation_error       | 400 | Битый синтаксис макроса в url / UTM                                                     |
| password_valivadion_error    | 400 | Пароль должен быть длиной от 4 до 128 символов                                          |
| recursive_redirect_error     | 400 | url указывает на домен из `short.base_domains` или его поддомен (рекурсивный редирект)  |
| long_url_unsafe_error        | 400 | (legacy) sync-блок; сейчас Unsafe обрабатывается async → `url_unsafe`                   |
| short_name_validation_error  | 400 | Ошибка валидации шаблона для short_name                                                 |
| tag_validation_error         | 400 | Ошибка в названии tag                                                                   |
| validation_error             | 400 | Прочие ошибки валидации (password+CAPTCHA вместе, schedule, subdomain+custom_domain, …) |
| auth_error                   | 401 | Не авторизован                                                                          |
| permission_denied_error      | 403 | Недостаточно прав у API key                                                             |
| api_key_scope_mismatch_error | 403 | API key привязан к другому scope                                                        |
| short_name_not_payed_error   | 402 | Вы имеете уже максимум коротких ссылок для вашей подписки                               |
| scope_not_found_error        | 404 | Не найден scope или нет к нему доступа                                                  |
| subdomain_not_found_error    | 404 | Не найден subdomain                                                                     |
| short_name_conflict_error    | 409 | Уже существует короткая ссылка с таким short_name в рамках данного subdomain            |
| short_name_exhausted_error   | 409 | Все возможные варианты short_name по шаблону уже заняты (для шаблонов с random_suffix)  |
| server_error                 | 500 | Внутренняя ошибка сервера                                                               |

---

<details open>
<summary><b>Пример запроса</b></summary>

```json
{
  // обязательное поле, ≥1 элемент, Σ weight = 100
  "targets": [
    {
      "url": "https://www.example.com/landing-a",
      "weight": 70,
      // необязательное поле, по умолчанию без лимита
      "max_clicks": 4000,
      // необязательное поле, по умолчанию без CPC
      "cpc": "12.50",
      // необязательное поле; требует cpc (cpc без budget — ок)
      "budget": "3500.00",
      // необязательное поле, по умолчанию null
      "utm": {
        "platform": "yandex_direct",
        "required": true,
        "source": "yandex",
        "medium": "cpc",
        "campaign": "{campaign_id}",
        "content": "{ad_id}",
        "term": "{keyword}"
      }
    },
    {
      "url": "https://www.example.com/landing-b",
      "weight": 30,
      "cpc": "8.00",
      "budget": "1500.00"
    }
  ],
  // необязательное поле, по умолчанию генерируется рандомный slug из 8 символов
  "short_name": "custom-slug/{random_suffix=8}",
  // необязательное поле, по умолчанию пустое
  "description": "This is a short link for my long URL",
  // необязательное поле, по умолчанию пустое
  "folder_id": "abc12345-e89b-12d3-a456-426614174000",
  // необязательное поле, по умолчанию пустое
  "subdomain": "my-short-link",
  // необязательное поле, по умолчанию пустое
  "set_password": "longPassword4488",
  // необязательное поле, по умолчанию пустое
  "expiration_time": "2024-12-31T23:59:59Z",
  // необязательное поле, по умолчанию пустое
  "beginning_time": "2024-06-01T12:00:00Z",
  // необязательное поле, по умолчанию пустое
  "tags": [
    "tag1",
    "tag2"
  ],
  // необязательное поле, по умолчанию 302
  "redirect_type": 301,
  // необязательное поле, по умолчанию без лимита (глобальный по short);
  // если задан — должен быть ≥ суммы targets[].max_clicks
  "max_clicks": 10000,
  // необязательное поле, по умолчанию без бюджета (глобальный по short);
  // если задан — должен быть ≥ суммы targets[].budget
  "budget": "5000.00",
  // необязательное поле, по умолчанию RUB
  "currency": "RUB",
  // необязательное поле, по умолчанию false
  "is_captcha": false
}
```

> `is_active` / `inactive_reason` клиент **не передаёт** — сервер считает сам
> (сроки, лимиты, …). В ответе оба поля read-only.
</details>

<details open>
<summary><b>Пример ответа</b></summary>

```json
{
  "short": {
    "id": 10000001,
    "short_name": "custom-slug/abc12345",
    "description": "This is a short link for my long URL",
    "folder_id": "abc12345-e89b-12d3-a456-426614174000",
    "subdomain": "my-short-link",
    "redirect_type": 301,
    "max_clicks": 10000,
    "clicks_count": 0,
    "budget": "5000.00",
    "spent": "0.00",
    "currency": "RUB",
    "targets": [
      {
        "id": 1,
        "url": "https://www.example.com/landing-a",
        "weight": 70,
        "max_clicks": 4000,
        "clicks_count": 0,
        "cpc": "12.50",
        "budget": "3500.00",
        "spent": "0.00",
        "is_active": true,
        "position": 0,
        "utm": {
          "platform": "yandex_direct",
          "required": true,
          "source": "yandex",
          "medium": "cpc",
          "campaign": "{campaign_id}",
          "content": "{ad_id}",
          "term": "{keyword}"
        }
      },
      {
        "id": 2,
        "url": "https://www.example.com/landing-b",
        "weight": 30,
        "max_clicks": null,
        "clicks_count": 0,
        "cpc": "8.00",
        "budget": "1500.00",
        "spent": "0.00",
        "is_active": true,
        "position": 1,
        "utm": null
      }
    ],
    "is_captcha": false,
    "is_active": true,
    "inactive_reason": null,
    "is_archived": false,
    "expiration_time": "2024-12-31T23:59:59Z",
    "beginning_time": "2024-06-01T12:00:00Z",
    "created_at": "2024-06-01T12:00:00Z",
    "updated_at": "2024-06-01T12:00:00Z"
  }
}
```

</details>
