<a id="get-support-admin-tickets"></a>

### <span style="background:#7CB342;padding:5px">GET</span> `/support/admin/tickets`

|                | Описание                                                                      |
|----------------|-------------------------------------------------------------------------------|
| **Назначение** | Админская очередь тикетов, отсортированная по приоритету подписки             |
| **Логика**     | 1. Роль `admin` или service token бота.                                       |
|                | 2. `ORDER BY priority_score DESC, created_at ASC`.                            |
|                | 3. Фильтры status / subscription.                                             |

---

| Kind            | Код | Описание     |
|-----------------|-----|--------------|
|                 | 200 | Очередь      |
| auth_error      | 401 | Не авторизован |
| forbidden_error | 403 | Не admin     |
| server_error    | 500 | Ошибка       |

---

<details open>
<summary><b>Пример ответа</b></summary>

```json
{
  "tickets": [
    {
      "id": "…",
      "subject": "Бюджет CPC",
      "status": "awaiting_admin",
      "priority_score": 1500,
      "subscription_snapshot": "BUSINESS_PLUS",
      "user_email": "agency@example.com",
      "created_at": "2026-07-21T10:00:00Z"
    },
    {
      "id": "…",
      "subject": "Не работает редирект",
      "priority_score": 400,
      "subscription_snapshot": "PRO",
      "user_email": "dev@example.com",
      "created_at": "2026-07-21T09:00:00Z"
    }
  ]
}
```

</details>
