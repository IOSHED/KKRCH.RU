<a id="get-shorts"></a>

### <span style="background:#7CB342;padding:5px">GET</span> `/shorts/{scope_id:int}`

|                | Описание                                                              |
|----------------|-----------------------------------------------------------------------|
| **Назначение** | Получает всю папочную структуру со всеми ссылками                     |
| **Auth**       | Bearer или X-Api-Key                                                  |
| **Логика**     | 1. Получает всю папочную структуру по переданной папке                |
| **Параметры**  | `scope_id:int` - идентификатор scope                                  |
| **Кеш**        | `shorts:list:{scope}:{v}:{filters_hash}`, TTL 30-60 сек (cache-aside) |

---

| Ответ                          | Код | Описание                               |
|--------------------------------|-----|----------------------------------------|
|                                | 200 | Успешно получено                       |
| validation_error               | 400 | Ошибка валидации всех полей            |
| custom_domain_validation_error | 400 | Ошибка валидации `custom_domain`       |
| auth_error                     | 401 | Не авторизован                         |
| permission_denied_error        | 403 | Недостаточно прав у API key            |
| api_key_scope_mismatch_error   | 403 | API key привязан к другому scope       |
| scope_not_found_error          | 404 | Не найден scope или нет к нему доступа |
| server_error                   | 500 | Внутренняя ошибка сервера              |

---

<details open>
<summary><b>Пример запроса</b></summary>

**Query параметры (все опциональные):**

- `redirect_type`: 301 или 302 (по умолчанию `null`, нет фильтрации)
- `short_name`: текстовый поиск по short_name, может включать `{random_suffix=8}`
- `target_url`: текстовый поиск по `targets[].url`
- `folder_name`: текстовый поиск по folder_name
- `subdomain`: фильтр по subdomain
- `custom_domain`: фильтр по собственному домену (FQDN)
- `folder_id`: UUID папки
- `tags`: список тегов (передается повторяющимся параметром)
- `tags_mode`: `any` или `all` (по умолчанию `any`)
- `with_password`: `true`/`false`
- `with_captcha`: `true`/`false`
- `with_expiration`: `true`/`false`
- `with_beginning`: `true`/`false`
- `with_max_clicks`: `true`/`false`
- `with_budget`: `true`/`false` (есть глобальный или per-target budget)
- `ad_platform`: фильтр по `utm.platform` у хотя бы одного target
- `is_active`: `true`/`false`
- `is_archived`: `true`/`false` (по умолчанию `false`)

</details>

<details open>
<summary><b>Пример ответа</b></summary>

```json
{
  "shorts": [
    {
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
      "clicks_count": 120,
      "budget": "5000.00",
      "spent": "1500.00",
      "currency": "RUB",
      "targets": [
        {
          "id": 1,
          "url": "https://www.example.com/a",
          "weight": 70,
          "max_clicks": null,
          "clicks_count": 84,
          "cpc": "12.50",
          "budget": "3500.00",
          "spent": "1050.00",
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
          "url": "https://www.example.com/b",
          "weight": 30,
          "clicks_count": 36,
          "cpc": "8.00",
          "budget": "1500.00",
          "spent": "450.00",
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
    }
  ]
}
```

</details>
