<a id="delete-company-scopes-invites"></a>

### <span style="background:#EF5350;padding:5px">DELETE</span> `/company/scopes/{scope_id:int}/invites/{invite_id:uuid}`

|                | Описание                                                    |
|----------------|-------------------------------------------------------------|
| **Назначение** | Отозвать pending-приглашение                                |
| **Логика**     | 1. Bearer: owner.                                           |
|                | 2. `revoked_at = now()` или DELETE; нет pending → 404.      |
|                | 3. Освобождает seat; in-app у invitee не удаляем (дешёво).  |
| **Параметры**  | `scope_id`, `invite_id` — path                              |
| **Auth**       | 🔒 Bearer owner                                             |

---

| Kind                  | Код | Описание            |
|-----------------------|-----|---------------------|
|                       | 204 | Отозвано            |
| auth_error            | 401 | Не авторизован      |
| scope_forbidden_error | 403 | Не владелец         |
| invite_not_found      | 404 | Нет pending invite  |
| server_error          | 500 | Внутренняя ошибка   |
