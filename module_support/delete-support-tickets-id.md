<a id="delete-support-tickets-id"></a>

### <span style="background:#E53935;padding:5px">DELETE</span> `/support/tickets/{ticket_id:uuid}`

|                | Описание                                                                 |
|----------------|--------------------------------------------------------------------------|
| **Назначение** | Пользователь удаляет обращение (soft-delete)                             |
| **Логика**     | 1. Ownership. 2. `deleted_at = now()`; скрывается из списков.            |
|                | 3. Закрытые и открытые — оба можно удалить; открытый снимается с очереди.|

Hard-delete только у admin (`DELETE /support/admin/tickets/{id}`).

---

| Kind                   | Код | Описание       |
|------------------------|-----|----------------|
|                        | 204 | Удалено        |
| auth_error             | 401 | Не авторизован |
| ticket_not_found_error | 404 | Нет доступа    |
| server_error           | 500 | Ошибка         |

---
