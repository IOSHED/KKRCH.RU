<a id="post-company-invites-accept"></a>

### <span style="background:#42A5F5;padding:5px">POST</span> `/company/invites/{token:str}/accept`

|                | Описание                                                                |
|----------------|-------------------------------------------------------------------------|
| **Назначение** | Принять приглашение (кнопка «Подключиться» в email / in-app)            |
| **Логика**     | 1. Bearer: текущий пользователь.                                        |
|                | 2. Находит invite по `SHA-256(token)`; нет / revoked / expired → 404.   |
|                | 3. Email профиля ≠ email invite → 403 `invite_email_mismatch_error`.    |
|                | 4. Уже member → 200 идемпотентно (текущие permissions).                 |
|                | 5. Seats (на момент accept) исчерпаны → 402 `seats_limit_error`.        |
|                | 6. INSERT `scope_members`, mark invite accepted, cache member.          |
|                | 7. Инвалидация `scopes:list` кеша invitee.                              |
| **Параметры**  | `token:str` — plaintext из письма/payload уведомления                   |
| **Auth**       | 🔒 Bearer invitee                                                       |

---

| Kind                        | Код | Описание                          |
|-----------------------------|-----|-----------------------------------|
|                             | 200 | Принято                           |
| auth_error                  | 401 | Не авторизован                    |
| invite_email_mismatch_error | 403 | Email аккаунта не совпал          |
| seats_limit_error           | 402 | Места закончились к моменту accept|
| invite_not_found            | 404 | Нет / истекло / отозвано          |
| server_error                | 500 | Внутренняя ошибка                 |

---

<details open>
<summary><b>Пример ответа 200</b></summary>

```json
{
  "scope_id": 10000001,
  "company_id": 1001,
  "permissions": [
    "shorts:read",
    "shorts:write",
    "folders:read",
    "folders:write",
    "stats:read"
  ]
}
```

</details>
