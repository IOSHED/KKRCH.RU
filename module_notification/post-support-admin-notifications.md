<a id="post-support-admin-notifications"></a>

### <span style="background:#42A5F5;padding:5px">POST</span> `/support/admin/notifications`

|                | Описание                                                                      |
|----------------|-------------------------------------------------------------------------------|
| **Назначение** | Кампания рассылки (релиз / system / promo) — batch fan-out job                |
| **Логика**     | 1. Service token. 2. INSERT campaign `queued`.                                |
|                | 3. Worker батчами по 500; UNIQUE (campaign_id, user_id).                      |
|                | 4. `promo` → только users с `in_app_promo=true`.                              |
|                | 5. Лимит 5 кампаний / сутки.                                                  |

---

| Kind            | Код | Описание           |
|-----------------|-----|--------------------|
|                 | 202 | В очереди          |
| validation_error| 400 | audience           |
| auth_error      | 401 | Нет token          |
| too_many_requests_error | 429 | Лимит кампаний |
| server_error    | 500 | Ошибка             |

---

<details open>
<summary><b>Пример — релиз (paid)</b></summary>

```json
{
  "kind": "release",
  "title": "Релиз 1.4",
  "body": "Доступен transfer на PERSONAL+.",
  "payload": { "version": "1.4.0" },
  "audience": "paid"
}
```

</details>
