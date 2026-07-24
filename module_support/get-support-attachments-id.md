<a id="get-support-attachments-id"></a>

### <span style="background:#7CB342;padding:5px">GET</span> `/support/attachments/{attachment_id:uuid}`

|                | Описание                                                              |
|----------------|-----------------------------------------------------------------------|
| **Назначение** | Скачать вложение с ACL (защита от IDOR)                               |
| **Логика**     | 1. Bearer: user владеет `ticket_id` вложения **или** service token.   |
|                | 2. Stream WebP; `Cache-Control: private, max-age=300`.                |
|                | 3. Чужой id → **404** (не 403).                                       |

---

| Kind         | Код | Описание       |
|--------------|-----|----------------|
|              | 200 | Файл           |
| auth_error   | 401 | Не авторизован |
|              | 404 | Нет / чужой    |
| server_error | 500 | Ошибка         |

---
