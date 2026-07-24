<a id="post-support-tickets-id-close"></a>

### <span style="background:#42A5F5;padding:5px">POST</span> `/support/tickets/{ticket_id:uuid}/close`

|                | Описание                                                          |
|----------------|-------------------------------------------------------------------|
| **Назначение** | Пользователь закрывает своё обращение                             |
| **Логика**     | 1. Ownership. 2. `status=closed`, `closed_by=user`.               |
|                | 3. Уведомляет бота (тикет снят с очереди).                        |

---

| Kind                   | Код | Описание        |
|------------------------|-----|-----------------|
|                        | 200 | Закрыто         |
| auth_error             | 401 | Не авторизован  |
| ticket_not_found_error | 404 | Нет доступа     |
| ticket_closed_error    | 409 | Уже закрыт      |
| server_error           | 500 | Ошибка          |

---

<details open>
<summary><b>Пример ответа</b></summary>

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "closed",
  "closed_by": "user",
  "closed_at": "2026-07-21T16:00:00Z"
}
```

</details>
