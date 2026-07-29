<a id="get-scopes-api-keys"></a>

### <span style="background:#7CB342;padding:5px">GET</span> `/api-keys/{scope_id:int}`

|                | Описание                                                         |
|----------------|------------------------------------------------------------------|
| **Назначение** | Список активных API keys scope (без secret)                      |
| **Логика**     | 1. Bearer: владелец `scope_id`.                                  |
|                | 2. Читает из Postgres ключи с `revoked_at IS NULL`.              |
|                | 3. Возвращает метаданные: `id`, `name`, `prefix`, `permissions`, |
|                | `created_at`, `last_used_at`.                                    |
| **Параметры**  | `scope_id:int` — path                                            |
| **Auth**       | 🔒 Bearer владельца                                              |

---

| Kind                  | Код | Описание                      |
|-----------------------|-----|-------------------------------|
|                       | 200 | Список ключей                 |
| auth_error            | 401 | Не авторизован                |
| scope_not_found_error | 404 | Scope не найден / нет доступа |
| server_error          | 500 | Внутренняя ошибка             |

---

<details open>
<summary><b>Пример ответа 200</b></summary>

```json
{
  "api_keys": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "name": "ci-ads-bot",
      "prefix": "a1b2c3d4",
      "permissions": "*",
      "scope_id": 10000001,
      "created_at": "2026-07-28T12:00:00Z",
      "last_used_at": "2026-07-28T15:30:00Z"
    },
    {
      "id": "661f9511-f3ac-52e5-b827-557766551111",
      "name": "partner-stats",
      "prefix": "9z8y7x6w",
      "permissions": [
        "shorts:write",
        "shorts:read",
        "stats:read"
      ],
      "scope_id": 10000001,
      "created_at": "2026-07-20T09:00:00Z",
      "last_used_at": null
    }
  ],
  "limit": 5,
  "active_count": 2
}
```

</details>

> `limit` = значение `api_keys.max_per_scope` из conf (удобно для UI).
> Отозванные ключи в список **не** входят.
