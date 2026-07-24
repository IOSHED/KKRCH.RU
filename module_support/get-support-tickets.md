<a id="get-support-tickets"></a>

### <span style="background:#7CB342;padding:5px">GET</span> `/support/tickets`

|                | Описание                                              |
|----------------|-------------------------------------------------------|
| **Назначение** | Список обращений текущего пользователя                |
| **Логика**     | 1. Bearer. 2. Фильтр `status`, пагинация.             |
|                | 3. Без soft-deleted (`deleted_at IS NULL`).           |

---

| Kind         | Код | Описание        |
|--------------|-----|-----------------|
|              | 200 | Список          |
| auth_error   | 401 | Не авторизован  |
| server_error | 500 | Ошибка          |

---

<details open>
<summary><b>Query</b></summary>

- `status`: `awaiting_admin` \| `awaiting_user` \| `closed` \| `all`
- `limit` / `cursor`

</details>

<details open>
<summary><b>Пример ответа</b></summary>

```json
{
  "tickets": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "subject": "Не работает редирект с паролем",
      "status": "awaiting_user",
      "priority_score": 400,
      "updated_at": "2026-07-21T15:00:00Z",
      "unread_admin_replies": 1
    }
  ],
  "next_cursor": null
}
```

</details>
