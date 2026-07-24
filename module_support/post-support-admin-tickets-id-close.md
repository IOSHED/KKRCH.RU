<a id="post-support-admin-tickets-id-close"></a>

### <span style="background:#42A5F5;padding:5px">POST</span> `/support/admin/tickets/{ticket_id:uuid}/close`

|                | Описание                                                          |
|----------------|-------------------------------------------------------------------|
| **Назначение** | Админ закрывает обращение                                         |
| **Логика**     | 1. `status=closed`, `closed_by=admin`.                            |
|                | 2. Уведомление пользователю `kind=support_reply` / system.        |

---

| Kind                   | Код | Описание       |
|------------------------|-----|----------------|
|                        | 200 | Закрыто        |
| auth_error             | 401 | Не авторизован |
| forbidden_error        | 403 | Не admin       |
| ticket_not_found_error | 404 | Нет тикета     |
| server_error           | 500 | Ошибка         |

---
