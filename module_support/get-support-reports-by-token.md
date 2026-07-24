<a id="get-support-reports-by-token"></a>

### <span style="background:#7CB342;padding:5px">GET</span> `/support/reports/by-token`

|                | Описание                                                              |
|----------------|-----------------------------------------------------------------------|
| **Назначение** | Статус гостевой жалобы + одно текстовое уточнение (magic-link)        |
| **Логика**     | 1. Token → hash lookup; TTL 14 дней.                                  |
|                | 2. GET: статус `is_reviewed`, `admin_action`.                         |
|                | 3. POST body `{ "note": "…" }` (≤2 KB) — одно уточнение (флаг).       |

Query: `?token=` (одноразовый session после первого GET можно не требовать).

---

| Kind                  | Код | Описание        |
|-----------------------|-----|-----------------|
|                       | 200 | Статус          |
| token_invalid_error   | 401 | Нет / истёк     |
| validation_error      | 400 | note слишком длинный |
| server_error          | 500 | Ошибка          |

---

<details open>
<summary><b>Пример ответа</b></summary>

```json
{
  "id": "7c9e6679-…",
  "kind": "abuse_link",
  "is_reviewed": false,
  "admin_action": null,
  "can_add_note": true,
  "created_at": "2026-07-21T14:00:00Z"
}
```

</details>
