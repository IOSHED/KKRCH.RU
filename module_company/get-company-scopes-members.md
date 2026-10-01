<a id="get-company-scopes-members"></a>

### <span style="background:#7CB342;padding:5px">GET</span> `/company/scopes/{scope_id:int}/members`

|                | Описание                                                              |
|----------------|-----------------------------------------------------------------------|
| **Назначение** | Список участников scope и pending-приглашений (для UI «Команда»)      |
| **Логика**     | 1. Bearer: owner **или** member этого scope.                          |
|                | 2. Lazy-expire: pending с `expires_at < now` не возвращаются          |
|                | &nbsp;&nbsp;&nbsp;(опционально DELETE expired).                       |
|                | 3. `members[]` — accepted (+ display_name, email, permissions).       |
|                | 4. `invites[]` — pending (id, email, permissions, created_at,         |
|                | &nbsp;&nbsp;&nbsp;expires_at). Token **не** отдаётся. Только owner.   |
|                | 5. `seats`: `{ used, max }` по компании (personal до convert:         |
|                | &nbsp;&nbsp;&nbsp;used=1+pending этого scope, max из тарифа owner).   |
|                | 6. `can_manage: bool` — invite/patch/remove доступны только owner.    |
| **Параметры**  | `scope_id:int` — path                                                 |
| **Auth**       | 🔒 Bearer owner или member                                            |

---

| Kind                  | Код | Описание                  |
|-----------------------|-----|---------------------------|
|                       | 200 | Список                    |
| auth_error            | 401 | Не авторизован            |
| scope_forbidden_error | 403 | Не owner и не member      |
| scope_not_found       | 404 | Scope не найден           |
| server_error          | 500 | Внутренняя ошибка         |

---

<details open>
<summary><b>Пример ответа 200</b></summary>

```json
{
  "scope_id": 10000001,
  "company_id": 1001,
  "seats": { "used": 3, "max": 10 },
  "can_manage": true,
  "members": [
    {
      "user_id": "550e8400-e29b-41d4-a716-446655440000",
      "email": "anna@example.com",
      "display_name": "Анна",
      "permissions": ["shorts:read", "shorts:write", "folders:read", "stats:read"],
      "joined_at": "2026-09-01T10:00:00Z"
    }
  ],
  "invites": [
    {
      "id": "660e8400-e29b-41d4-a716-446655440001",
      "email": "boris@example.com",
      "permissions": "*",
      "created_at": "2026-09-28T12:00:00Z",
      "expires_at": "2026-10-05T12:00:00Z"
    }
  ]
}
```

</details>

> Владелец в `members` **не** дублируется: UI показывает его отдельно как
> owner с полным доступом.
