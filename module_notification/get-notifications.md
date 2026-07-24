<a id="get-notifications"></a>

### <span style="background:#7CB342;padding:5px">GET</span> `/notifications`

|                | Описание                                                    |
|----------------|-------------------------------------------------------------|
| **Назначение** | Inbox уведомлений текущего пользователя                     |
| **Логика**     | 1. Bearer. 2. Список `user_notifications` DESC (`limit` ≤100, default 50). |
|                | 3. Query: `since` (timestamptz), `limit`. Фильтров `unread_only`/`kind` нет. |

---

| Kind         | Код | Описание       |
|--------------|-----|----------------|
|              | 200 | Inbox          |
| auth_error   | 401 | Не авторизован |
| server_error | 500 | Ошибка         |

---

<details open>
<summary><b>Пример ответа</b></summary>

```json
{
  "notifications": [
    {
      "id": "…",
      "kind": "support_reply",
      "title": "Ответ поддержки",
      "body": "Проверьте fingerprint…",
      "payload": { "ticket_id": "550e8400-…" },
      "is_read": false,
      "created_at": "2026-07-21T15:00:01Z"
    },
    {
      "id": "…",
      "kind": "release",
      "title": "Релиз 1.4",
      "body": "Доступен модуль transfer…",
      "payload": { "version": "1.4.0" },
      "is_read": true,
      "created_at": "2026-07-20T09:00:00Z"
    }
  ]
}
```

</details>
