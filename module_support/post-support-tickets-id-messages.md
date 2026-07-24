<a id="post-support-tickets-id-messages"></a>

### <span style="background:#42A5F5;padding:5px">POST</span> `/support/tickets/{ticket_id:uuid}/messages`

|                | Описание                                                              |
|----------------|-----------------------------------------------------------------------|
| **Назначение** | Ответ пользователя в открытый тикет                                   |
| **Логика**     | 1. Ownership; статус ≠ `closed`.                                      |
|                | 2. INSERT message; `status → awaiting_admin`.                         |
|                | 3. Уведомляет бота.                                                   |

---

| Kind                   | Код | Описание              |
|------------------------|-----|-----------------------|
|                        | 201 | Сообщение создано     |
| validation_error       | 400 | Пустой body           |
| auth_error             | 401 | Не авторизован        |
| ticket_closed_error    | 409 | Тикет закрыт          |
| ticket_not_found_error | 404 | Нет доступа           |
| server_error           | 500 | Ошибка                |

---

<details open>
<summary><b>Пример запроса</b></summary>

```json
{
  "body": "User-Agent тот же, всё равно 401."
}
```

</details>
