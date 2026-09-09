<a id="get-support-tickets-id"></a>

### <span style="background:#7CB342;padding:5px">GET</span> `/support/tickets/{ticket_id:uuid}`

|                | Описание                                                         |
|----------------|------------------------------------------------------------------|
| **Назначение** | Карточка тикета + лента сообщений + метаданные вложений          |
| **Логика**     | 1. Ownership assert (свой тикет) или admin.                      |
|                | 2. Closed старше `closed_ticket_retention` → hard-delete + 404.  |
|                | 3. Messages ASC (`is_internal=false` для user).                  |
|                | 4. Admin/service видит и internal notes.                         |

---

| Kind                   | Код | Описание           |
|------------------------|-----|--------------------|
|                        | 200 | Тикет              |
| auth_error             | 401 | Не авторизован     |
| ticket_not_found_error | 404 | Нет / чужой        |
| server_error           | 500 | Ошибка             |

---

<details open>
<summary><b>Пример ответа (сокращённо)</b></summary>

```json
{
  "ticket": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "subject": "Не работает редирект с паролем",
    "status": "awaiting_user",
    "priority_score": 400,
    "closed_by": null
  },
  "messages": [
    {
      "id": "…",
      "author_type": "user",
      "body": "После ввода пароля…",
      "attachments": [],
      "created_at": "2026-07-21T14:05:00Z"
    },
    {
      "id": "…",
      "author_type": "admin",
      "body": "Проверьте User-Agent на refresh.",
      "attachments": [
        { "id": "…", "url": "/api/v1/support/attachments/…", "content_type": "image/webp" }
      ],
      "created_at": "2026-07-21T15:00:00Z"
    }
  ]
}
```

</details>
