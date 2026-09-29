<a id="post-support-admin-tickets-id-attachments"></a>

### <span style="background:#42A5F5;padding:5px">POST</span> `/support/admin/tickets/{ticket_id:uuid}/attachments`

|                | Описание                                                                      |
|----------------|-------------------------------------------------------------------------------|
| **Назначение** | Upload изображения к сообщению оператора (сжатие на backend → WebP)           |
| **Логика**     | 1. Service token; статус ≠ `closed`. Query `message_id` — к какому сообщению. |
|                | 2. Без `message_id` — последнее **публичное** admin-сообщение.                |
|                | 3. Явный `message_id` — только `author_type=admin`, `is_internal=false`.      |
|                | 4. Max **2 MB** raw, **3** файла / сообщение; resize 1280 / WebP q=70.        |
|                | 5. `user_id` вложения = владелец тикета (user может скачать через ACL).       |
|                | 6. Без push боту (оператор сам загружает).                                    |

Клиент после `POST …/admin/tickets/{id}/messages` передаёт `message_id` из ответа.

---

| Kind                       | Код | Описание           |
|----------------------------|-----|--------------------|
|                            | 201 | Сохранено          |
| validation_error           | 400 | Не image / size    |
| too_many_attachments_error | 400 | > 3 на сообщение   |
| auth_error                 | 401 | Нет token          |
| ticket_not_found_error     | 404 | Нет тикета         |
| ticket_closed_error        | 409 | Тикет закрыт       |
| server_error               | 500 | Ошибка             |

---
