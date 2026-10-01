<a id="delete-company-scopes-members"></a>

### <span style="background:#EF5350;padding:5px">DELETE</span> `/company/scopes/{scope_id:int}/members/{user_id:uuid}`

|                | Описание                                                              |
|----------------|-----------------------------------------------------------------------|
| **Назначение** | Удалить участника из scope (owner) или выйти самому (`user_id = me`)  |
| **Логика**     | 1. Bearer.                                                            |
|                | 2. Разрешено: owner **или** `user_id == auth.user_id` (self-leave).   |
|                | 3. DELETE из `scope_members`; нет строки → 404.                       |
|                | 4. Инвалидация member-cache; scope пропадает из `GET /scopes` у user. |
|                | 5. Удаление **владельца компании** этим путём запрещено → 409.        |
| **Параметры**  | `scope_id`, `user_id` — path                                          |
| **Auth**       | 🔒 Bearer owner или self                                              |

---

| Kind                  | Код | Описание                         |
|-----------------------|-----|----------------------------------|
|                       | 204 | Удалён                           |
| auth_error            | 401 | Не авторизован                   |
| scope_forbidden_error | 403 | Не owner и не self               |
| member_not_found      | 404 | Не участник                      |
| owner_remove_error    | 409 | Нельзя удалить владельца company |
| server_error          | 500 | Внутренняя ошибка                |

---

> Тела ответа нет (`204 No Content`).
