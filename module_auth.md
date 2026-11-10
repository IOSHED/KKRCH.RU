# Auth Module

Модуль для авторизации пользователей и управления токенами доступа.
Один email (без учёта регистра) — один пользователь. Новый способ входа
с той же почтой привязывается к уже существующему аккаунту.

Способы входа:

- [Яндекс ID](https://oauth.yandex.ru/) и [Google](https://developers.google.com/identity/protocols/oauth2) —
  authorization code
- [VK ID](https://id.vk.com/) — authorization code + PKCE (`code_verifier`, `device_id`, `state`)
- Email + 6-значный код — без пароля

Провайдер обязан отдать email. Пустой email — `400`, аккаунт не создаётся.

Microsoft в этой версии нет. Пароля нет.

> **Дорожная карта:** для полного соответствия
> [ФЗ-149](https://eais.rkn.gov.ru/docs/149.pdf) /
> [ФЗ-152](https://72.rkn.gov.ru/p21978/p25026/p25030/) возможны SMS;
> для корпоративного доступа — SSO.

---

## Оглавление

| Иконка | Раздел                                | Ссылка                                                   |
|--------|---------------------------------------|----------------------------------------------------------|
| 🧭     | Информация по реализации              | [ссылка](#информация-по-реализации)                      |
| 🔐     | Модель токенов                        | [ссылка](#модель-токенов)                                |
| 🧬     | Fingerprint и защита от кражи токенов | [ссылка](#fingerprint-и-защита-от-кражи-токенов)         |
| 🗄️    | Хранилища данных                      | [ссылка](#хранилища-данных)                              |
| 🔁     | Консистентность Postgres ↔ Redis      | [ссылка](#консистентность-postgres--redis)               |
| ⚙️     | Конфигурация                          | [ссылка](#конфигурация)                                  |
| 📋     | Сводная таблица эндпоинтов            | [ссылка](#сводная-таблица-эндпоинтов)                    |
| 🔗     | Эндпоинты                             | [ссылка](#сводная-таблица-эндпоинтов)                    |
| ↳      | POST /auth/oauth_login                | [ссылка](module_auth/post-auth-oauth_login.md)           |
| ↳      | POST /auth/email/code                 | [ссылка](module_auth/post-auth-email-code.md)            |
| ↳      | POST /auth/email/verify               | [ссылка](module_auth/post-auth-email-verify.md)          |
| ↳      | POST /auth/refresh                    | [ссылка](module_auth/post-auth-refresh.md)               |
| ↳      | GET /auth/subscription_plans          | [ссылка](module_auth/get-auth-subscription_plans.md)     |
| ↳      | GET /auth/profile                     | [ссылка](module_auth/get-auth-profile.md)                |
| ↳      | PATCH /auth/profile                   | [ссылка](module_auth/patch-auth-profile.md)              |
| ↳      | GET /auth/sessions                    | [ссылка](module_auth/get-auth-sessions.md)               |
| ↳      | DELETE /auth/sessions/{session_id}    | [ссылка](module_auth/delete-auth-sessions-session_id.md) |
| ↳      | POST /auth/logout                     | [ссылка](module_auth/post-auth-logout.md)                |
| ↳      | POST /auth/logout_all                 | [ссылка](module_auth/post-auth-logout_all.md)            |

---

## Информация по реализации

Сессия авторизации хранится в виде токена `Authorization: Bearer <токен>`,  
где `<токен>` — stateful uuid в `Redis` c TTL, информацией:

```json
{
  "user_id": "UUID",
  "session_id": "UUID",
  "token_id": "UUID",
  "subscription": "FREE",
  "roles": []
}
```

Для консистентности данных в Postgres и Redis реализован [Session Watcher Service](../session_watcher.md)
(в т.ч. lifecycle подписки / cooling — см. [Payment](../privat_http_api/module_payment.md)).

Типы подписки (см. также [subscription_politics](../../business/subscription_politics.md)):

| Subscription  | Описание                                                               |
|---------------|------------------------------------------------------------------------|
| FREE          | Бесплатный доступ для ознакомления                                     |
| FREE_PLUS     | Расширенный бесплатный тариф (публичный каталог на запуске, `is_view`) |
| PERSONAL      | Индивидуальное использование                                           |
| PRO           | Создатели, фрилансеры                                                  |
| BUSINESS      | Малый бизнес / ИП (корп.)                                              |
| BUSINESS_PLUS | Растущие команды (корп.)                                               |

---

## Модель токенов

Система использует **stateful opaque-токены** (UUID), не JWT.

```mermaid
graph LR
    subgraph "Клиент хранит"
        AT[access_token<br/>UUID]
        RT[refresh_token<br/>UUID]
    end

    subgraph "Redis хранит"
        AE["access:{uuid}<br/>→ AccessTokenEntry"]
        RE["refresh:{uuid}<br/>→ RefreshTokenEntry"]
    end

    AT -.->|lookup| AE
    RT -.->|lookup| RE
```

- **Access token** — короткоживущий (по умолчанию 15 мин prod, 8 ч dev), передаётся в заголовке
  `Authorization: Bearer <uuid>`.
- **Refresh token** — долгоживущий (по умолчанию 31 день), нужен для получения новой пары токенов.
- При каждом `/auth/refresh` старый refresh-токен инвалируется, выдается новый (**rolling rotation**).

---

## Fingerprint и защита от кражи токенов

При создании refresh-токена сервер вычисляет **fingerprint-хеш** из:

- `User-Agent`
- `X-Fingerprint` (опционально, генерируется клиентом один раз)

При каждом `/auth/refresh` сервер пересчитывает fingerprint и сравнивает с сохранённым в Redis.  
В случае несовпадения — токен уничтожается и доступ блокируется.

```mermaid
sequenceDiagram
    participant Атакующий
    participant S as Сервер
    participant R as Redis
    Атакующий ->> S: POST /auth/refresh<br/>(украденный refresh_token, другой User-Agent)
    S ->> R: GET refresh:{id}
    R -->> S: RefreshTokenEntry
    S ->> S: fingerprint ≠ stored hash
    S ->> R: DEL refresh:{id}
    S -->> Атакующий: 401 Unauthorized
    Note over R: Токен уничтожен, оба (злоумышленник и жертва) теряют доступ
```

---

## Хранилища данных

### PostgreSQL — долгосрочное хранение

| Таблица              | Назначение                                                                                                                                                                                     |
|----------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `users`              | Профиль пользователя (id, email, display_name, subscription, roles; billing: `subscription_ends_at`, `cooling_until`, `cooling_enforced_at` — [Payment](../privat_http_api/module_payment.md)) |
| `oauth_accounts`     | Способы входа пользователя. Уникальна пара `(provider, provider_user_id)`. У одного `user_id` может быть несколько провайдеров: `yandex`, `google`, `vk`, `email`                              |
| `subscription_plans` | Справочник тарифных планов с лимитами; `is_view` — видимость в GET `/auth/subscription_plans`                                                                                                  |
| `sessions`           | Аудит сессий, ревокация, revocation_reason                                                                                                                                                     |

### Redis — оперативный кеш токенов

| Ключ                              | Содержимое                                                              | TTL              |
|-----------------------------------|-------------------------------------------------------------------------|------------------|
| `access:{token_id}`               | AccessTokenEntry (user_id, session_id, subscription, roles)             | acces_ttl        |
| `refresh:{token_id}`              | RefreshTokenEntry (user_id, session_id, fingerprint_hash, subscription) | refresh_ttl      |
| `refresh_index:{user_id}`         | SET всех refresh-токенов пользователя                                   | —                |
| `auth:email_code:{sha256(email)}` | Хеш 6-значного кода                                                     | `email_code.ttl` |
| `auth:email_send:{sha256(email)}` | Флаг «письмо уже отправлено», повтор молчит                             | `resend_after`   |

Код письма пишется в Redis сразу (не через deferred pipeline): он должен
существовать до отправки письма и не связан с транзакцией Postgres.
В Redis лежит SHA-256 кода, не сам код.

> `subscription` и `roles` кешируются в access token при логине — auth middleware

---

## Консистентность Postgres ↔ Redis

Паттерн **deferred Redis writes**:

1. Все операции с Postgres выполняются в транзакции.
2. Операции с Redis откладываются (defer) в очередь.
3. После успешного `tx.commit()` выполняются queued Redis-записи.
4. Если транзакция откатывается — очередь удаляется, орфанные данные не остаются.

> Для устранения расхождений работает сервис [Session Watcher](../session_watcher.md), который сверяет списки сессий
> между Redis и Postgres.

### Поведение при потере данных Redis

| Данные                    | Источник истины        | Fallback при miss Redis                                                           | Примечание                                                                                        |
|---------------------------|------------------------|-----------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------|
| Access token              | Redis (`access:{id}`)  | **Postgres** — `sessions.current_access_token_id` + JOIN `users`                  | Middleware [`auth.rs`](../../../services/backend/src/views/http_api/src/actix/middleware/auth.rs) |
| Refresh token             | Redis (`refresh:{id}`) | **Нет** — payload (fingerprint, subscription) только в Redis                      | После eviction нужен повторный login                                                              |
| `refresh_index:{user_id}` | Redis SET              | **Частично** — Session Watcher пересобирает индекс из существующих `refresh:{id}` | Не восстанавливает сами токены                                                                    |
| `sessions.expires_at`     | Postgres               | —                                                                                 | Обновляется при login и каждом `/auth/refresh` (rolling TTL)                                      |
| Rate limit / ban          | Redis                  | Fail-open (запрос пропускается)                                                   | Осознанная деградация                                                                             |
| `short:resolve:*`         | Postgres               | Cache miss → SELECT                                                               | Корректно, медленнее                                                                              |
| Bloom availability        | Postgres               | BF miss / отсутствие BF → SELECT                                                  | Корректно; ложноположительный BF → лишний SELECT                                                  |

**Практический вывод:** при полной очистке Redis уже выданные **access**-токены продолжают работать до истечения access
TTL (или session `expires_at` в PG). **Refresh** и **logout_all** по evicted refresh-токенам — нет; пользователь
re-login после истечения access.

---

## Конфигурация

Файл: `conf/local.yaml` или `prod.yaml`

```yaml
auth:
  acces_ttl: 15m      # Время жизни access-токена
  refresh_ttl: 31d    # Время жизни refresh-токена
  lock_duration: 15m  # Базовая длительность блокировки
  user_agent: "..."   # User-Agent для OAuth-запросов

  yandex:
    client_id: "..."
    client_secret: "..."

  google:
    client_id: "..."
    client_secret: "..."

  vk:
    client_id: ""          # id приложения VK ID
    service_token: ""      # сервисный ключ, если приложение конфиденциальное

  email_code:
    ttl: 10m
    resend_after: 60s
    max_attempts: 5
```

Пустой `vk.client_id` выключает VK ID: запрос получает `400 validation_error`.

Переменные окружения (префикс `APP__`, вложенность через `__`):

| Переменная                            | Назначение           |
|---------------------------------------|----------------------|
| `APP__AUTH__VK__CLIENT_ID`            | ID приложения VK ID  |
| `APP__AUTH__VK__SERVICE_TOKEN`        | Сервисный ключ VK ID |
| `APP__AUTH__EMAIL_CODE__TTL`          | Например `10m`       |
| `APP__AUTH__EMAIL_CODE__RESEND_AFTER` | Например `60s`       |
| `APP__AUTH__EMAIL_CODE__MAX_ATTEMPTS` | Например `5`         |

Почта с кодом уходит сразу через `mail.smtp` / `mail.file`, не через очередь
`mail-worker`. При выключенной почте (`mail.enabled: false`) ручка всё равно
отвечает `200`, но письма нет.

---

## Сводная таблица эндпоинтов

| Метод    | Путь                       | Авторизация | Описание                                                                         |
|----------|----------------------------|-------------|----------------------------------------------------------------------------------|
| `POST`   | `/auth/oauth_login`        | —           | Яндекс, Google, VK ID. Тот же email привязывает провайдера. Пустой email — `400` |
| `POST`   | `/auth/email/code`         | —           | Письмо с 6-значным кодом. Всегда `200`, если email корректен                     |
| `POST`   | `/auth/email/verify`       | —           | Вход по коду из письма                                                           |
| `POST`   | `/auth/refresh`            | —           | Обновление пары токенов                                                          |
| `GET`    | `/auth/subscription_plans` | —           | Справочник тарифных планов                                                       |
| `GET`    | `/auth/profile`            | 🔒 Bearer   | Получение профиля                                                                |
| `PATCH`  | `/auth/profile`            | 🔒 Bearer   | Обновление профиля                                                               |
| `GET`    | `/auth/sessions`           | 🔒 Bearer   | Список активных сессий                                                           |
| `DELETE` | `/auth/sessions/{id}`      | 🔒 Bearer   | Отзыв конкретной сессии                                                          |
| `POST`   | `/auth/logout`             | 🔒 Bearer   | Выход (текущая сессия)                                                           |
| `POST`   | `/auth/logout_all`         | 🔒 Bearer   | Выход (все сессии)                                                               |
