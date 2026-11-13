<a id="patch-auth-profile"></a>

### <span style="background:#FB8C00;padding:5px">PATCH</span> `/auth/profile`

|                | Описание                                                                                  |
|----------------|-------------------------------------------------------------------------------------------|
| **Назначение** | Частичное обновление профиля текущего пользователя                                        |
| **Логика**     | 1. Валидирует access_token в заголовке `Authorization`.                                   |
|                | 2. Принимает необязательные `display_name` (1–100 символов) и `locale` (`ru`/`en`/`de`/`fr`). Нужно хотя бы одно поле. |
|                | 3. Обновляет переданные поля и фиксирует новое `updated_at`. `locale` — язык исходящих писем, не язык ответа. |
|                | 4. Возвращает обновлённый профиль пользователя.                                           |

---

| Kind                 | Код | Описание                                   |
|----------------------|-----|--------------------------------------------|
|                      | 200 | Профиль обновлён, возвращает `user`        |
| validation_error     | 400 | Пустое тело, `display_name` пустой или длиннее 100 символов, либо `locale` вне `ru`/`en`/`de`/`fr` |
| auth_error           | 401 | Нет или некорректный access token          |
| user_not_found_error | 404 | Пользователь не найден                     |
| server_error         | 500 | Внутренняя ошибка сервера                  |

---

<details open>
<summary><b>Пример запроса</b></summary>

```json
{
  "display_name": "Иван Иванов",
  "locale": "ru"
}
```

</details>

<details open>
<summary><b>Пример ответа</b></summary>

```json
{
  "user": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "auth_provider": "YANDEX",
    "email": "user@example.com",
    "display_name": "Иван Иванов",
    "locale": "ru",
    "subscription": "FREE",
    "roles": [],
    "created_at": "2025-01-15T10:00:00Z",
    "updated_at": "2025-01-17T12:30:00Z"
  }
}
```

</details>
