<a id="delete-company-scopes-locks"></a>

### <span style="background:#EF5350;padding:5px">DELETE</span> `/company/scopes/{scope_id:int}/locks/shorts/{short_id:int}`

|                | Описание                                                      |
|----------------|---------------------------------------------------------------|
| **Назначение** | Снять soft-lock (закрытие редактора / после save)             |
| **Логика**     | 1. Bearer: holder. Чужой lock не трогаем → 409.               |
|                | 2. Нет lock → 204 идемпотентно.                               |
|                | 3. DEL Redis + Pub `short_unlock`.                            |
| **Параметры**  | `scope_id`, `short_id` — path                                 |
| **Auth**       | 🔒 Bearer holder                                              |

---

| Kind                    | Код | Описание            |
|-------------------------|-----|---------------------|
|                         | 204 | Снято / не было     |
| auth_error              | 401 | Не авторизован      |
| scope_forbidden_error   | 403 | Нет доступа         |
| short_edit_locked_error | 409 | Lock у другого user |
| server_error            | 500 | Внутренняя ошибка   |
