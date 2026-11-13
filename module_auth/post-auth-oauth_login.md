<a id="post-auth-oauth_login"></a>

### <span style="background:#42A5F5;padding:5px">POST</span> `/auth/oauth_login`

|                | Описание                                                                      |
|----------------|-------------------------------------------------------------------------------|
| **Назначение** | Вход или регистрация через Яндекс ID, Google или VK ID                         |
| **Логика**     | 1. Сервер обменивает authorization code на access_token провайдера.           |
|                | 2. Получает профиль (email, name, id). Для VK ID нужны `code_verifier`, `device_id`, `state`. |
|                | 3. Связка provider+id уже есть — сессия этого пользователя (`200`).           |
|                | 4. Email свободен — новый пользователь (`201`), `users.locale` берётся из `Accept-Language` запроса (`ru`/`en`/`de`/`fr`, иначе `en`). Уже существующий email — провайдер привязывается к тому же аккаунту (`200`), `locale` не меняется. |
|                | 5. Email пустой — `400`, аккаунт не создаётся.                                |
|                | 6. Сессия: Postgres + access/refresh в Redis.                                 |
| **Заголовки**  | `User-Agent`, `X-Fingerprint` (опционально)                                   |

---

| Kind                 | Код | Описание                                         |
|----------------------|-----|--------------------------------------------------|
|                      | 200 | Существующий пользователь (в том числе после привязки нового провайдера) |
|                      | 201 | Зарегистрирован новый пользователь               |
| validation_error     | 400 | Ошибка валидации, VK без PKCE, пустой email или провайдер не настроен |
| server_error         | 500 | Внутренняя ошибка сервера                        |

---

<details open>
<summary><b>Пример запроса</b></summary>

```json
{
  "code": "4/0AQSTgQF...xyz",
  "provider": "GOOGLE",
  "redirect_uri": "https://example.com/home"
}
```

`provider`: `YANDEX` | `GOOGLE` | `VK`. Для `VK` обязательны `code_verifier`, `device_id` и `state`.
`MAX` и `EMAIL` сюда не передаются: для них отдельные ручки.

Поля `code_verifier`, `device_id`, `state` опциональны в JSON. Яндекс и Google их игнорируют.

</details>

<details>
<summary><b>Схема</b></summary>

```mermaid
sequenceDiagram
    participant C as Клиент
    participant S as Сервер
    participant Y as Яндекс OAuth
    participant PG as PostgreSQL
    participant R as Redis
    C ->> Y: Авторизация у провайдера
    Y -->> C: authorization code
    C ->> S: POST /auth/oauth_login { provider, code, redirect_uri }
    S ->> Y: Получить access_token по code
    Y -->> S: access_token
    S ->> Y: GET /info
    Y -->> S: { email, name, id }
    alt OAuth-аккаунт найден
        S ->> PG: Найти пользователя по oauth_accounts
        S -->> C: 200 { user, tokens }
    else Новый пользователь
        S ->> PG: INSERT users + oauth_accounts
        S -->> C: 201 { user, tokens, is_new_user: true }
    end
    S ->> PG: INSERT sessions, COMMIT
    S ->> R: SET access + refresh
```

</details>

<details open>
<summary><b>Пример ответа</b></summary>

```json
{
  "tokens": {
    "access_token": "string",
    "expires_in": 900,
    "refresh_token": "string",
    "token_type": "bearer"
  },
  "user": {
    "id": "uuid",
    "auth_provider": "YANDEX",
    "email": "user@example.com",
    "display_name": "John",
    "locale": "en",
    "subscription": "FREE",
    "roles": [
      "RoleFromCompany"
    ],
    "created_at": "2025-01-15T10:00:00Z",
    "updated_at": "2025-01-17T10:00:00Z"
  },
  "is_new_user": true
}
```

</details>
