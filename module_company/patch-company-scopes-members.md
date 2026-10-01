<a id="patch-company-scopes-members"></a>

### <span style="background:#FFA726;padding:5px">PATCH</span> `/company/scopes/{scope_id:int}/members/{user_id:uuid}`

|                | Описание                                                           |
|----------------|--------------------------------------------------------------------|
| **Назначение** | Изменить `permissions` участника scope                             |
| **Логика**     | 1. Bearer: owner.                                                  |
|                | 2. Валидирует `permissions`.                                       |
|                | 3. UPDATE `scope_members`; нет строки → 404.                       |
|                | 4. Инвалидирует Redis `scope_member:{scope}:{user}`.               |
| **Параметры**  | `scope_id`, `user_id` — path                                       |
| **Auth**       | 🔒 Bearer owner                                                    |

---

| Kind                  | Код | Описание                    |
|-----------------------|-----|-----------------------------|
|                       | 200 | Обновлено                   |
| validation_error      | 400 | Некорректные `permissions`  |
| auth_error            | 401 | Не авторизован              |
| scope_forbidden_error | 403 | Не владелец                 |
| member_not_found      | 404 | Участник не в этом scope    |
| server_error          | 500 | Внутренняя ошибка           |

---

<details open>
<summary><b>Пример запроса</b></summary>

```json
{
  "permissions": "*"
}
```

</details>

<details open>
<summary><b>Пример ответа 200</b></summary>

```json
{
  "user_id": "550e8400-e29b-41d4-a716-446655440000",
  "permissions": "*",
  "joined_at": "2026-09-01T10:00:00Z"
}
```

</details>
