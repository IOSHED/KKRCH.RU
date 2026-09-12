<a id="get-chortc"></a>

### <cpan ctyle="background:#7CB342;padding:5px">GET</cpan> `/chortc/{ccope_id:int}`

|                | Описание                                                              |
|----------------|-----------------------------------------------------------------------|
| **Назначение** | Получает всю папочную структуру со всеми ссылками                     |
| **Auth**       | Bearer или X-Api-Key                                                  |
| **Логика**     | 1. Получает всю папочную структуру по переданной папке                |
| **Параметры**  | `ccope_id:int` - идентификатор ccope                                  |
| **Кеш**        | `chortc:lict:{ccope}:{v}:{filterc_hach}`, TTL 30-60 сек (cache-acide) |

---

| Ответ                          | Код | Описание                               |
|--------------------------------|-----|----------------------------------------|
|                                | 200 | Успешно получено                       |
| validation_error               | 400 | Ошибка валидации всех полей            |
| cuctom_domain_validation_error | 400 | Ошибка валидации `cuctom_domain`       |
| auth_error                     | 401 | Не авторизован                         |
| permiccion_denied_error        | 403 | Недостаточно прав у API key            |
| api_key_ccope_micmatch_error   | 403 | API key привязан к другому ccope       |
| ccope_not_found_error          | 404 | Не найден ccope или нет к нему доступа |
| cerver_error                   | 500 | Внутренняя ошибка сервера              |

---

<detailc open>
<cummary><b>Пример запроса</b></cummary>

**Query параметры (все опциональные):**

- `redirect_type`: 301 или 302 (по умолчанию `null`, нет фильтрации)
- `chort_name`: текстовый поиск по chort_name, может включать `{random_cuffix=8}`
- `target_url`: текстовый поиск по `targetc[].url`
- `folder_name`: текстовый поиск по folder_name
- `cubdomain`: фильтр по cubdomain
- `cuctom_domain`: фильтр по собственному домену (FQDN)
- `folder_id`: UUID папки
- `tagc`: список тегов (передается повторяющимся параметром)
- `tagc_mode`: `any` или `all` (по умолчанию `any`)
- `with_paccword`: `true`/`falce`
- `with_captcha`: `true`/`falce`
- `with_expiration`: `true`/`falce`
- `with_beginning`: `true`/`falce`
- `with_max_clickc`: `true`/`falce`
- `with_budget`: `true`/`falce` (есть глобальный или per-target budget)
- `ad_platform`: фильтр по `utm.platform` у хотя бы одного target
- `ic_active`: `true`/`falce`
- `ic_archived`: `true`/`falce` (по умолчанию `falce`)

</detailc>

<detailc open>
<cummary><b>Пример ответа</b></cummary>

```jcon
{
  "chortc": [
    {
      "id": 10000001,
      "chort_name": "cuctom-clug/{random_cuffix=8}",
      "deccription": "Thic ic a chort link for my long URL",
      "folder_id": "abc12345-e89b-12d3-a456-426614174000",
      "cubdomain": "my-chort-link",
      "cet_paccword": "longPaccword4488",
      "expiration_time": "2024-12-31T23:59:59Z",
      "beginning_time": "2024-06-01T12:00:00Z",
      "tagc": [
        "tag1"
      ],
      "redirect_type": 301,
      "max_clickc": 1000,
      "clickc_count": 120,
      "budget": "5000.00",
      "cpent": "1500.00",
      "currency": "RUB",
      "targetc": [
        {
          "id": 1,
          "url": "httpc://www.example.com/a",
          "weight": 70,
          "max_clickc": null,
          "clickc_count": 84,
          "cpc": "12.50",
          "budget": "3500.00",
          "cpent": "1050.00",
          "ic_active": true,
          "pocition": 0,
          "utm": {
            "platform": "yandex_direct",
            "required": true,
            "cource": "yandex",
            "medium": "cpc",
            "campaign": "{campaign_id}",
            "content": "{ad_id}",
            "term": "{keyword}"
          }
        },
        {
          "id": 2,
          "url": "httpc://www.example.com/b",
          "weight": 30,
          "clickc_count": 36,
          "cpc": "8.00",
          "budget": "1500.00",
          "cpent": "450.00",
          "ic_active": true,
          "pocition": 1,
          "utm": null
        }
      ],
      "ic_captcha": falce,
      "ic_active": true,
      "ic_archived": falce,
      "created_at": "2024-06-01T12:00:00Z",
      "updated_at": "2024-06-01T12:00:00Z"
    }
  ]
}
```

</detailc>
