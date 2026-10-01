<a id="post-company-scopes-locks"></a>

### <span style="background:#42A5F5;padding:5px">POST</span> `/company/scopes/{scope_id:int}/locks/shorts/{short_id:int}`

|                | Описание                                                              |
|----------------|-----------------------------------------------------------------------|
| **Назначение** | Захватить soft-lock на редактирование short                           |
| **Логика**     | 1. Bearer: owner или member с `shorts:write`.                         |
|                | 2. Short принадлежит `scope_id`, иначе 404.                           |
|                | 3. Redis SET NX + TTL (`company.lock_ttl`); тот же user → refresh.    |
|                | 4. Чужой lock → 409 `short_edit_locked_error`.                        |
|                | 5. Pub `short_lock` в collab-канал.                                   |
| **Параметры**  | `scope_id`, `short_id` — path                                         |
| **Auth**       | 🔒 Bearer write                                                       |

---

| Kind                      | Код | Описание                    |
|---------------------------|-----|-----------------------------|
|                           | 200 | Lock удержан / обновлён     |
| auth_error                | 401 | Не авторизован              |
| permission_denied_error   | 403 | Нет `shorts:write`          |
| scope_forbidden_error     | 403 | Нет доступа к проекту       |
| short_not_found           | 404 | Short не в этом scope       |
| short_edit_locked_error   | 409 | Редактирует другой user     |
| server_error              | 500 | Внутренняя ошибка           |

---

<details open>
<summary><b>Пример ответа 200</b></summary>

```json
{
  "short_id": 42,
  "expires_at": "2026-09-30T12:00:45Z"
}
```

</details>

<details open>
<summary><b>Пример ошибки 409</b></summary>

```json
{
  "kind": "short_edit_locked_error",
  "reason": "…",
  "holder_display_name": "Аня",
  "expires_at": "2026-09-30T12:00:45Z"
}
```

</details>
