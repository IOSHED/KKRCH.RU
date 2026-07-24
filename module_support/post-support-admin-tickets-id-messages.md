<a id="post-support-admin-tickets-id-messages"></a>

### <span style="background:#42A5F5;padding:5px">POST</span> `/support/admin/tickets/{ticket_id:uuid}/messages`

|                | Описание                                                                      |
|----------------|-------------------------------------------------------------------------------|
| **Назначение** | Ответ оператора (бот) или internal note                                       |
| **Логика**     | 1. Service token. 2. INSERT message.                                          |
|                | 3. Если `is_internal=false` → `awaiting_user` + `user_notifications` + Redis. |
|                | 4. Если `is_internal=true` → не уведомлять user, не менять status.            |
|                | 5. Audit log.                                                                 |

---

| Kind                   | Код | Описание     |
|------------------------|-----|--------------|
|                        | 201 | Записано     |
| auth_error             | 401 | Нет token    |
| ticket_closed_error    | 409 | Закрыт       |
| ticket_not_found_error | 404 | Нет тикета   |
| server_error           | 500 | Ошибка       |

---

<details open>
<summary><b>Пример</b></summary>

```json
{
  "body": "Проверьте fingerprint при refresh.",
  "is_internal": false
}
```

</details>
