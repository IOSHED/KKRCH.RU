<a id="patch-notifications-id-read"></a>

### <span style="background:#FB8C00;padding:5px">PATCH</span> `/notifications/{notification_id:uuid}/read`

|                | Описание                                      |
|----------------|-----------------------------------------------|
| **Назначение** | Пометить уведомление прочитанным              |
| **Логика**     | Ownership; `is_read = true` (идемпотентно).   |

---

| Kind                         | Код | Описание       |
|------------------------------|-----|----------------|
|                              | 200 | Обновлено      |
| auth_error                   | 401 | Не авторизован |
| notification_not_found_error | 404 | Нет записи     |
| server_error                 | 500 | Ошибка         |

---
