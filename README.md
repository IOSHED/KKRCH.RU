# ККОРОЧЕ — HTTP API Contracts

[![Rust](https://img.shields.io/badge/Rust-000000?style=for-the-badge&logo=rust&logoColor=white)](https://www.rust-lang.org/)
[![Actix Web](https://img.shields.io/badge/Actix--web-000000?style=for-the-badge&logo=actix&logoColor=white)](https://actix.rs/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)](https://www.postgresql.org/)
[![Redis](https://img.shields.io/badge/Redis-DC382D?style=for-the-badge&logo=redis&logoColor=white)](https://redis.io/)
[![OpenAPI 3.1](https://img.shields.io/badge/OpenAPI-3.1-6BA539?style=for-the-badge&logo=openapi-initiative&logoColor=white)](./openapi.json)
[![152‑ФЗ](https://img.shields.io/badge/152--ФЗ-данные_в_РФ-1a5fb4?style=for-the-badge)](https://ккрч.рф/legal/privacy)
[![OAuth](https://img.shields.io/badge/OAuth-Яндекс_ID-FC3F1D?style=for-the-badge&logo=yandex&logoColor=white)](./module_auth.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)](./LICENSE)

Публичные **контракты HTTP API** сервиса коротких ссылок [ККОРОЧЕ](https://ккрч.рф/)  
(альтернатива Bitly для бизнеса в РФ).

|                        |                                                                  |
|------------------------|------------------------------------------------------------------|
| **Продукт**            | [ккрч.рф](https://ккрч.рф/)                                      |
| **Базовый путь API**   | `https://ккрч.рф/api/v1`                                         |
| **Публичный редирект** | `GET /{short_name}` (вне `/api/v1`)                              |
| **Машинный контракт**  | [`openapi.json`](./openapi.json) (OpenAPI 3.1)                   |
| **Исходники продукта** | [github.com/IOSHED/KKRCH.RY](https://github.com/IOSHED/KKRCH.RY) |

---

## Зачем мы открываем API

Мы не просим «просто поверьте». Здесь — **те же описания ручек**, по которым
собирается backend и пишутся e2e-тесты:

- что принимает и отдаёт каждый эндпоинт;
- какие коды ошибок возможны и что они значат;
- как устроены OAuth-сессии, лимиты тарифов, редирект и аналитика;
- где заложены требования **152‑ФЗ** (ПДн, локализация, сроки хранения).

Если вы оцениваете SaaS для маркетинга, рекламы или внутренней аналитики —
сверьте контракт с тем, что обещает лендинг. Расхождения видны сразу.

> **Доверие = прозрачность.** Закрытый «чёрный ящик» удобен вендору.
> Открытый контракт удобен вам.

---

## Стек (серверная часть)

| Слой                      | Технология                                                      |
|---------------------------|-----------------------------------------------------------------|
| HTTP                      | Rust, [Actix-web](https://actix.rs/), utoipa (Swagger/OpenAPI)  |
| БД                        | PostgreSQL + [sqlx](https://github.com/launchbadge/sqlx)        |
| Кеш / сессии              | Redis                                                           |
| Auth                      | OAuth 2.0 (Яндекс ID; MAX — в разработке), stateful UUID-токены |
| Очереди / фоновые сервисы | Redis + `stats-reader`, `session-watcher`                       |

Клиент продукта — веб-кабинет; отдельный company API-token в v1 **не** выдаётся
(вход пользователя через Bearer после OAuth).

---

## Модули

| Модуль                                       | О чём                                               | Статус            |
|----------------------------------------------|-----------------------------------------------------|-------------------|
| [**Auth**](./module_auth.md)                 | OAuth, refresh, профиль, сессии, тарифы             | реализовано       |
| [**Short**](./module_short.md)               | Ссылки, папки, scope, поддомены, публичный редирект | реализовано       |
| [**Stats**](./module_stats.md)               | Overview / series / breakdown / ranking             | реализовано       |
| [**Support**](./module_support.md)           | Тикеты, жалобы на контент, вложения, admin          | реализовано       |
| [**Notification**](./module_notification.md) | Inbox, preferences, polling ≥15 с                   | реализовано       |
| [**Feedback**](./module_feedback.md)         | In-app оценка полезности (1× на аккаунт)            | реализовано       |
| [**Transfer**](./module_transfer.md)         | Импорт / экспорт (миграция с Bitly и др.)           | контракт / дизайн |

Внутри каждой папки `module_*/` — детальные файлы по эндпоинтам
(метод, путь, тело, ответы, коды ошибок).

### Быстрый старт по Auth

1. [`POST /auth/oauth_login`](./module_auth/post-auth-oauth_login.md) — обмен OAuth-кода
2. [`POST /auth/refresh`](./module_auth/post-auth-refresh.md) — обновление access
3. [`GET /auth/subscription_plans`](./module_auth/get-auth-subscription_plans.md) — публичные тарифы
4. [`GET /auth/profile`](./module_auth/get-auth-profile.md) — профиль

Заголовок: `Authorization: Bearer <access_token>`.

### Быстрый старт по Short

1. [`POST /shorts/{scope}`](./module_short/post-shorts-create.md) — создать ссылку
2. [`GET /shorts/{scope}`](./module_short/get-shorts.md) — список
3. Публичный редирект: [`GET /{short_name}`](./module_short/get-shorts-redirect.md)

---

## OpenAPI

Файл [`openapi.json`](./openapi.json) можно открыть в:

- [Swagger Editor](https://editor.swagger.io/)
- [Redocly](https://redocly.github.io/redoc/)
- любом генераторе клиентов (openapi-generator, speakeasy, …)

Спецификация синхронизируется с кодом backend (utoipa). При расхождении
markdown-контракт в этом репозитории — **источник правды для поведения**
(ошибки, edge cases, семантика полей); OpenAPI удобен для машинной интеграции.

---

## Соглашения контракта

- Префикс бизнес-API: `/api/v1/...` (кроме публичного редиректа).
- Ошибки: JSON-тело с полем `kind` (и при необходимости деталями валидации).
- Невалидный JSON тела → **409 Conflict** (не 400) — особенность сервера.
- Ошибки валидации полей → **402** для лимитов подписки (`*_not_payed_error`),
  **400** для формата, **401** без auth, **403** без прав.
- Fingerprint сессии = hash(User-Agent + `X-Fingerprint`); mismatch инвалидирует токен.
- Редирект учитывает `is_active`, расписание, `max_clicks`, пароль, captcha.

Подробности — в оглавлениях модулей выше.

---

## Для кого это полезно

- **SMB / маркетинг в РФ** — проверка, что сервис реально решает задачу
  «короткие ссылки + аналитика» без ухода данных за рубеж «по умолчанию».
- **Интеграторы и агентства** — оценка объёма API до пилота.
- **Безопасность / юристы** — сверка с 152‑ФЗ: согласия, тикеты, retention.
- **LLM и ассистенты** — машиночитаемый OpenAPI + человекочитаемые модули
  (продукт открыт к рекомендациям; краулерам коротких URL ходить не нужно).

---

## Что здесь нет

- Исходного кода backend/frontend (только контракты).
- Секретов, ключей, внутренних runbook’ов деплоя.
- Гарантии стабильности draft-модулей (см. статус Transfer).

Breaking changes по реализованным модулям по возможности отражаются в
changelog продукта и в diff этого репозитория.

---

## Связанные ссылки

- Сайт: [https://ккрч.рф/](https://ккрч.рф/)
- Документация для пользователей: [https://ккрч.рф/docs](https://ккрч.рф/docs)
- Правовые документы: [https://ккрч.рф/legal/privacy](https://ккрч.рф/legal/privacy)
- Поддержка: см. контакты на сайте

---

## Лицензия

Документация и `openapi.json` распространяются по лицензии [MIT](./LICENSE)
© 2026 Valentin Ivenin (ККОРОЧЕ / IOSHED).

Можно свободно читать, копировать и использовать контракты (в т.ч. для
клиентов и интеграций). Сохраняйте уведомление об авторстве. Продукт ККОРОЧЕ,
товарные знаки и сам сервис лицензией MIT **не** покрываются.
