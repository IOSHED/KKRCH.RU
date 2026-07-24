<a id="delete-support-admin-tickets-id"></a>

### <span style="background:#E53935;padding:5px">DELETE</span> `/support/admin/tickets/{ticket_id:uuid}`

|                | Описание                                                              |
|----------------|-----------------------------------------------------------------------|
| **Назначение** | Админ удаляет тикет (hard или soft по политике; v1 — hard cascade)    |
| **Логика**     | 1. Admin. 2. DELETE messages/attachments + ticket.                    |

---

| Kind                   | Код | Описание       |
|------------------------|-----|----------------|
|                        | 204 | Удалено        |
| auth_error             | 401 | Не авторизован |
| forbidden_error        | 403 | Не admin       |
| ticket_not_found_error | 404 | Нет тикета     |
| server_error           | 500 | Ошибка         |

---
