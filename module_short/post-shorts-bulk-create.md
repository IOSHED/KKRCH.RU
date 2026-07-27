<a id="post-shorts-bulk-create"></a>

### <span style="background:#42A5F5;padding:5px">POST</span> `/shorts/{scope_шв:int}/bulk`

|                      | Описание                                                                                                             |
|----------------------|----------------------------------------------------------------------------------------------------------------------|
| **Назначение**       | Создаёт укороченные ссылки массово                                                                                   |
| **Логика**           | 0. Если число элементов превышает лимит (`bulk.max_items`) → 400 `too_many_items_error`                              |
|                      | 1. Каждый элемент проходит независимую валидацию; невалидные → в `errors` с индексом                                 |
|                      | &nbsp;&nbsp;&nbsp;- `recursive_redirect_error` / `long_url_unsafe_error` / `targets_validation_error` — как у create |
|                      | 2. Для валидных элементов параллельно вычисляются Argon2-хеши паролей                                                |
|                      | 3. Последовательно разрешаются шаблоны `{random_suffix=N}` (sequential или random+retry)                             |
|                      | 4. Bulk INSERT shorts + short_targets в рамках одной транзакции                                                      |
|                      | 5. Конфликты `short_name` внутри batch → в `errors`; остальные ссылки не отменяются                                  |
|                      | 6. После commit инвалидируется Redis-версия scope                                                                    |
| **Параметры**        | `scope:int` - индификатор scope                                                                                      |
| **Инвалидация кеша** | `INCR shorts:scope:{scope}:v`                                                                                        |

---

| Ответ                       | Код | Описание                                                                                                  |
|-----------------------------|-----|-----------------------------------------------------------------------------------------------------------|
|                             | 201 | Созданы все короткие ссылки                                                                               |
|                             | 207 | Частично созданы короткие ссылки, в ответе будет информация о том, какие ссылки были созданы, а какие нет |
| too_many_items_error        | 400 | Превышен лимит элементов в bulk-запросе (`bulk.max_items`)                                                |
| long_url_validation_error   | 400 | Некорректный `targets[].url`                                                                              |
| targets_validation_error    | 400 | Пустой targets / weight / длина                                                                           |
| cpc_budget_validation_error | 400 | `target.budget` без `target.cpc`                                                                          |
| utm_validation_error        | 400 | `utm.required` без source/medium/campaign                                                                 |
| max_clicks_validation_error | 400 | Глобальный `max_clicks` < суммы `targets[].max_clicks`                                                    |
| budget_validation_error     | 400 | Глобальный `budget` < суммы `targets[].budget`                                                            |
| utm_validation_error        | 400 | Обязательные UTM не заполнены                                                                             |
| recursive_redirect_error    | 400 | url указывает на домен из `short.base_domains` или его поддомен (рекурсивный редирект)                    |
| password_valivadion_error   | 400 | Пароль должен быть длиной от 4 до 128 символов                                                            |
| long_url_unsafe_error       | 400 | url помечен Yandex Safe Browsing как опасный (malware, phishing и т.п.); проверка — batch Lookup API      |
| short_name_validation_error | 400 | Ошибка валидации шаблона для short_name                                                                   |
| tag_validation_error        | 400 | Ошибка в названии tag                                                                                     |
| validation_error            | 400 | Прочие ошибки валидации                                                                                   |
| auth_error                  | 401 | Не авторизован                                                                                            |
| short_name_not_payed_error  | 402 | Вы имеете уже максимум коротких ссылок для вашей подписки                                                 |
| scope_not_found_error       | 404 | Не найден scope или нет к нему доступа                                                                    |
| subdomain_not_found_error   | 404 | Не найден subdomain                                                                                       |
| short_name_conflict_error   | 409 | Уже существует короткая ссылка с таким short_name в рамках данного subdomain                              |
| short_name_exhausted_error  | 409 | Все возможные варианты short_name по шаблону уже заняты (для шаблонов с random_suffix)                    |
| server_error                | 500 | Внутренняя ошибка сервера                                                                                 |

---

<details open>
<summary><b>Пример запроса</b></summary>

```json
{
  "shorts": [
    {
      // обязательное поле, ≥1 элемент, Σ weight = 100
      "targets": [
        {
          "url": "https://www.example.com/some/very/long/url",
          "weight": 100
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
        "tag1"
      ],
      // необязательное поле, по умолчанию 302
      "redirect_type": 301,
      // необязательное поле, по умолчанию без лимита
      "max_clicks": 1000,
      // необязательное поле, по умолчанию false
      "is_captcha": false
    },
    {
      "targets": [
        {
          "url": "https://www.example.com/a",
          "weight": 50,
          "cpc": "5.00",
          "budget": "500.00",
          "utm": {
            "platform": "google_ads",
            "required": true,
            "source": "google",
            "medium": "cpc",
            "campaign": "{network}",
            "content": "{creative}",
            "term": "{keyword}"
          }
        },
        {
          "url": "https://www.example.com/b",
          "weight": 50,
          "cpc": "5.00",
          "budget": "500.00"
        }
      ],
      "short_name": "split-test",
      "budget": "1000.00",
      "currency": "RUB"
    }
  ]
}
```

</details>

<details open>
<summary><b>Пример ответа</b></summary>

```json
{
  "shorts": {
    "id": 10000001,
    "short_name": "custom-slug/{random_suffix=8}",
    "description": "This is a short link for my long URL",
    "folder_id": "abc12345-e89b-12d3-a456-426614174000",
    "subdomain": "my-short-link",
    "set_password": "longPassword4488",
    "expiration_time": "2024-12-31T23:59:59Z",
    "beginning_time": "2024-06-01T12:00:00Z",
    "tags": [
      "tag1"
    ],
    "redirect_type": 301,
    "max_clicks": 1000,
    "targets": [
      {
        "id": 1,
        "url": "https://www.example.com/some/very/long/url",
        "weight": 100,
        "clicks_count": 0,
        "spent": "0.00",
        "is_active": true,
        "position": 0,
        "utm": null
      }
    ],
    "is_captcha": false,
    "is_active": true,
    "is_archived": false,
    "created_at": "2024-06-01T12:00:00Z",
    "updated_at": "2024-06-01T12:00:00Z"
  },
  "errors": [
    {
      "index": 0,
      "kind": "subdomain_not_payed_error",
      "reason": "Subdomain 'my-short-link' requires premium subscription"
    }
  ]
}
```

> Может быть `shorts` полностью пустым, список ошибок `errors` всегда будет, кроме 500 ошибок.

</details>
