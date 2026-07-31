<a id="patch-shorts-bulk"></a>

### <span style="background:#FFA726;padding:5px">PATCH</span> `/shorts/bulk`

|                      | Описание                                                                                |
|----------------------|-----------------------------------------------------------------------------------------|
| **Назначение**       | Массово редактирует короткие ссылки                                                     |
| **Auth**             | Bearer или X-Api-Key                                                                    |
| **Логика**           | 0. Если число элементов превышает лимит (`bulk.max_items`) → 400 `too_many_items_error` |
|                      | 1. Каждый патч проходит независимую валидацию; невалидные → в `errors` с индексом       |
|                      | 2. Для элементов с `set_password` параллельно вычисляются Argon2-хеши                   |
|                      | 3. Если передан `targets` — **full replace** набора: с `id` обновление, без `id` create;|
|                      | &nbsp;&nbsp;&nbsp;отсутствующие id удаляются (клики: FK SET NULL)                       |
|                      | 4. Safe Browsing только для `targets[].url`, которые реально меняются / новые           |
|                      | 5. Bulk UPDATE только для ссылок текущего пользователя                                  |
|                      | 6. Не найденные ID → в `errors` с `kind = not_found`                                    |
|                      | 7. После commit инвалидируется Redis-версия scope + resolve keys                        |
| **Инвалидация кеша** | `INCR shorts:scope:{scope}:v`, drop `short:resolve:*` для затронутых                    |

---

| Ответ                       | Код | Описание                                                                                             |
|-----------------------------|-----|------------------------------------------------------------------------------------------------------|
|                             | 200 | Ссылка обновлена                                                                                     |
|                             | 207 | Частично обновлены ссылки, в ответе будет информация о том, какие ссылки были обновлены, а какие нет |
| too_many_items_error        | 400 | Превышен лимит элементов в bulk-запросе (`bulk.max_items`)                                           |
| long_url_validation_error   | 400 | Некорректный `targets[].url`                                                                         |
| targets_validation_error    | 400 | Пустой targets / weight / длина                                                                      |
| cpc_budget_validation_error | 400 | `target.budget` без `target.cpc`                                                                     |
| utm_validation_error        | 400 | `utm.required` без source/medium/campaign                                                            |
| max_clicks_validation_error | 400 | Глобальный `max_clicks` < суммы `targets[].max_clicks`                                               |
| budget_validation_error     | 400 | Глобальный `budget` < суммы `targets[].budget`                                                       |
| utm_validation_error        | 400 | Обязательные UTM не заполнены                                                                        |
| long_url_unsafe_error       | 400 | url помечен Yandex Safe Browsing как опасный; проверяется только если url меняется (batch)           |
| short_name_validation_error | 400 | Ошибка валидации шаблона для short_name                                                              |
| tag_validation_error        | 400 | Ошибка в названии tag                                                                                |
| validation_error            | 400 | Прочие ошибки валидации                                                                              |
| auth_error                  | 401 | Не авторизован                                                                                       |
| permission_denied_error       | 403 | Недостаточно прав у API key |
| api_key_scope_mismatch_error  | 403 | API key привязан к другому scope |
| subdomain_not_payed_error   | 402 | Запрошены премиум subdomain для ссылки без его имения                                                |
| short_name_not_payed_error  | 402 | Превышен лимит коротких ссылок по подписке                                                       |
| scope_not_found_error       | 404 | Не найден scope или нет к нему доступа                                                               |
| subdomain_not_found_error   | 404 | Не найден subdomain                                                                                  |
| short_name_conflict_error   | 409 | Уже существует короткая ссылка с таким short_name в рамках данного subdomain                         |
| server_error                | 500 | Внутренняя ошибка сервера                                                                            |

---

<details open>
<summary><b>Пример запроса</b></summary>

> Все поля необязательные, кроме `id`.
> Поля, которых нет в JSON — **не меняются**. Явный `null` **сбрасывает**
> nullable-поле (`budget`, `max_clicks`, `beginning_time`, `expiration_time`,
> `folder_id`, `subdomain`, `set_password`). `set_password: ""` тоже снимает пароль
> (совместимость с прежним контрактом).
> `is_active` в PATCH **нет** (read-only). `is_archived=true` → после recompute
> `is_active=false` с `inactive_reason=archived` (если нет более приоритетной
> причины: лимиты, сроки, subdomain).

```json
{
  "shorts": [
    {
      "id": 10000001,
      // необязательное поле — full replace targets, если передано
      "targets": [
        {
          "id": 1,
          "url": "https://www.example.com/a",
          "weight": 60,
          "cpc": "10.00",
          "budget": "1200.00",
          "utm": {
            "platform": "vk_ads",
            "required": true,
            "source": "vk_ads",
            "medium": "cpc",
            "campaign": "{{campaign_id}}",
            "content": "{{banner_id}}",
            "term": "{{geo}}_{{gender}}_{{age}}"
          }
        },
        {
          "url": "https://www.example.com/b-new",
          "weight": 40,
          "cpc": "8.00",
          "budget": "800.00"
        }
      ],
      "short_name": "custom-slug/{random_suffix=8}",
      "description": "This is a short link for my long URL",
      "folder_id": "abc12345-e89b-12d3-a456-426614174000",
      "subdomain": "my-short-link",
      "set_password": "longPassword4488",
      "expiration_time": "2024-12-31T23:59:59Z",
      "beginning_time": "2024-06-01T12:00:00Z",
      "tags": [
        "tag1",
        "tag2"
      ],
      "redirect_type": 301,
      "max_clicks": 1000,
      "budget": "2000.00",
      "currency": "RUB",
      "is_captcha": false,
      "is_archived": false
    },
    {
      "id": 10000002,
      "short_name": "custom-slug/{random_suffix=8"
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
    "budget": "2000.00",
    "spent": "0.00",
    "currency": "RUB",
    "targets": [
      {
        "id": 1,
        "url": "https://www.example.com/a",
        "weight": 60,
        "cpc": "10.00",
        "budget": "1200.00",
        "spent": "0.00",
        "clicks_count": 0,
        "is_active": true,
        "position": 0,
        "utm": {
          "platform": "vk_ads",
          "required": true,
          "source": "vk_ads",
          "medium": "cpc",
          "campaign": "{{campaign_id}}",
          "content": "{{banner_id}}",
          "term": "{{geo}}_{{gender}}_{{age}}"
        }
      },
      {
        "id": 2,
        "url": "https://www.example.com/b-new",
        "weight": 40,
        "cpc": "8.00",
        "budget": "800.00",
        "spent": "0.00",
        "clicks_count": 0,
        "is_active": true,
        "position": 1,
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
      "index": 1,
      "kind": "short_name_validation_error",
      "reason": "Invalid short_name format: missing closing '}' for random_suffix"
    }
  ]
}
```

> Так же может быть `shorts` полностью пустым, а `errors` будет содержать все переданные ссылки с ошибками, но при этом
> ответ все еще 207

</details>
