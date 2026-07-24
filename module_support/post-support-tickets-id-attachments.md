<a id="post-support-tickets-id-attachments"></a>

### <span style="background:#42A5F5;padding:5px">POST</span> `/support/tickets/{ticket_id:uuid}/attachments`

|                | Описание                                                                 |
|----------------|--------------------------------------------------------------------------|
| **Назначение** | Upload изображения (сжатие на backend → WebP)                            |
| **Логика**     | 1. Ownership; статус ≠ `closed`. Query `message_id` — к какому сообщению. |
|                | 2. Max **2 MB** raw, **3** файла / **это** message (не на весь тикет).   |
|                | 3. Resize max side **1280**, WebP q=70; dedup sha256.                    |
|                | 4. Push боту `ticket_attachment` (фото).                                 |
|                | 5. Download только через ACL [`GET …/attachments/{id}`](get-support-attachments-id.md). |

Без `message_id` вложение вешается на **последнее** user-сообщение. Клиент
после `POST …/messages` должен передавать `message_id` из ответа — иначе при
уже заполненном первом сообщении (3 скрина) ответ с фото даёт
`too_many_attachments_error`.

---

| Kind                       | Код | Описание           |
|----------------------------|-----|--------------------|
|                            | 201 | Сохранено          |
| validation_error           | 400 | Не image / size    |
| too_many_attachments_error | 400 | > 3 на сообщение   |
| auth_error                 | 401 | Не авторизован     |
| ticket_not_found_error     | 404 | Нет доступа        |
| ticket_closed_error        | 409 | Тикет закрыт       |
| server_error               | 500 | Ошибка             |

---
