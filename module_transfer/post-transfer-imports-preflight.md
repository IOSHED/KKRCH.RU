<a id="post-transfer-imports-preflight"></a>

### <span style="background:#42A5F5;padding:5px">POST</span> `/transfer/{scope_id:int}/imports/preflight`

|                | Описание                                                                                  |
|----------------|-------------------------------------------------------------------------------------------|
| **Назначение** | Синхронная проверка лимитов import **до** создания job                                    |
| **Auth**       | Bearer или X-Api-Key (`transfer_import`)                                                  |
| **Content-Type** | `application/json`                                                                      |
| **Логика**     | 1. Gate тарифа: `transfer_daily_bytes > 0` → иначе 402 `transfer_not_payed_error`.         |
|                | 2. Клиент передаёт **оценки** (сервер файл не сканирует): `would_create_shorts`,          |
|                | &nbsp;&nbsp;&nbsp;`unique_subdomains[]`, `raw_events_outside_retention`.                    |
|                | 3. Сверка с `max_shorts`, наличием subdomain в scope, retention.                          |
|                | 4. Успех → 200 + сводка; нарушение → 402/400, job не создаётся.                           |
| **Параметры**  | `scope_id:int` — path                                                                     |

> Preflight **не** дублируется автоматически на `POST …/imports`. Import create
> проверяет только paywall transfer, conflict и byte-quota. Лимиты shorts /
> subdomain / retention — ответственность клиента (вызвать preflight) или
> runner (построчные ошибки).

---

| Kind                         | Код | Описание                                              |
|------------------------------|-----|-------------------------------------------------------|
|                              | 200 | Preflight пройден                                     |
| stats_retention_error        | 400 | Raw вне retention при `ignore_retention_limit=false`  |
| validation_error             | 400 | Некорректное тело                                     |
| auth_error                   | 401 | Не авторизован                                        |
| permission_denied_error      | 403 | Недостаточно прав API key                             |
| api_key_scope_mismatch_error | 403 | API key привязан к другому scope                      |
| scope_not_found_error        | 404 | Scope недоступен                                      |
| short_name_not_payed_error   | 402 | `current + would_create > max_shorts`                 |
| subdomain_not_payed_error    | 402 | Subdomain из списка отсутствуют в scope               |
| transfer_not_payed_error     | 402 | `transfer_daily_bytes = 0`                            |
| server_error                 | 500 | Внутренняя ошибка                                     |

---

<details open>
<summary><b>Пример запроса</b></summary>

```json
{
  "would_create_shorts": 120,
  "unique_subdomains": ["promo", "go"],
  "raw_events_outside_retention": 0,
  "ignore_retention_limit": false
}
```

</details>

<details open>
<summary><b>Пример ответа 200</b></summary>

```json
{
  "ok": true,
  "subscription": "Personal",
  "limits": {
    "max_shorts": 300,
    "max_subdomains": 0,
    "stats_click_retention_days": 90
  },
  "import_summary": {
    "would_create_shorts": 120,
    "current_shorts": 45,
    "remaining_short_slots": 255,
    "unique_subdomains": ["promo", "go"],
    "missing_subdomains": [],
    "raw_events_outside_retention": 0
  }
}
```

`subscription` — Debug-формат enum (`Personal`, `BusinessPlus`, …).

</details>

<details>
<summary><b>Поля тела</b></summary>

| Поле | Тип | Default | Описание |
|------|-----|---------|----------|
| `would_create_shorts` | i64 | `0` | Сколько новых shorts создаст import |
| `unique_subdomains` | string[] | `[]` | Subdomain, которые встречаются в файле |
| `raw_events_outside_retention` | i64 | `0` | Число raw-событий старше retention |
| `ignore_retention_limit` | bool | `false` | `true` — не падать на outside retention |

</details>
