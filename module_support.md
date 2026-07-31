# Support Module

Модуль **обращений в поддержку** и жалоб на контент.

In-app inbox, preferences, polling и campaigns — в
[`module_notification`](module_notification.md).

**Принципы v1:**

1. **Один оператор** — человек с доступом к Telegram-боту (пароль) и/или к
   Postgres. Мульти-админ, assignment, роли — **не делаем**, пока не понадобится.
2. **152-ФЗ** — минимизация ПДн, явные цели, сроки хранения, локализация РФ,
   согласие, право на удаление (см. [152-ФЗ](#соответствие-152-фз)).
3. **Дешёвая эксплуатация** — агрессивное сжатие вложений, короткие TTL,
   batch fan-out.

> **Статус:** реализовано (HTTP API + Postgres + TG-бот). Ниже — контракт модуля.

---

## Оглавление

| Иконка | Раздел | Ссылка |
|--------|--------|--------|
| 🎯 | Use cases | [ссылка](#use-cases) |
| 👤 | Модель оператора (1 admin) | [ссылка](#модель-оператора-1-admin) |
| 🗄️ | Модель данных | [ссылка](#модель-данных) |
| ⚖️ | Соответствие 152-ФЗ | [ссылка](#соответствие-152-фз) |
| 💸 | Удешевление эксплуатации | [ссылка](#удешевление-эксплуатации) |
| 📬 | Очередь и приоритеты | [ссылка](#очередь-и-приоритеты) |
| 🚨 | Abuse → действие по ссылке | [ссылка](#abuse--действие-по-ссылке) |
| 🖼️ | Вложения | [ссылка](#вложения) |
| 🤖 | Admin bot | [ссылка](#admin-bot) |
| ⚙️ | Лимиты | [ссылка](#лимиты) |
| 📋 | Эндпоинты | [ссылка](#сводная-таблица-эндпоинтов) |

Endpoint-файлы: [`module_support/`](module_support/).

---

## Use cases

| Сценарий | Кто | Как |
|----------|-----|-----|
| Жалоба на phishing без аккаунта | Гость | `POST /support/reports` + consent |
| Уточнение / статус жалобы | Гость | magic-link на email |
| Тикет зарегистрированного | User | `POST /support/tickets` |
| Скриншоты | User | attachments (WebP, ACL) |
| Закрыть / reopen в окне TTL (7d) | User / Admin | `…/close`, `…/reopen` |
| Отметить жалобу рассмотренной | Admin (бот/SQL) | `is_reviewed` + optional deactivate short |
| Анонс релиза | Admin | см. [Notification](module_notification.md) |
| Ответ почти сразу в UI | User | см. [Notification](module_notification.md) |

---

## Модель оператора (1 admin)

**Admin = один человек**, который:

- знает пароль support-бота **или**
- имеет прямой доступ к БД / service token.

Нет таблицы «команда админов», нет broadcast «новый админ», нет assignee.

| Механизм | Назначение |
|----------|------------|
| `SUPPORT_ADMIN_PASSWORD` (hash в секретах) | Вход в Telegram-бот |
| `support_operator` (1 row) | Кеш сессии бота: telegram id, `authenticated_at`, `last_seen_at` |
| Service token бота | Вызовы `/support/admin/*` |

При `/logout` или смене пароля — обнулить `support_operator`, потребовать пароль снова.

Мульти-админ / assignment — вне v1 (отдельный RFC).

Admin HTTP API защищён **только** service token бота (и опционально ручной
SQL). Отдельная роль `users.roles = admin` в v1 **не обязательна**.

---

## Модель данных

### `support_reports` — гостевые жалобы

| Колонка | Тип | Описание |
|---------|-----|----------|
| `id` | UUID PK | |
| `email` | TEXT | ПДн; для ответа |
| `kind` | ENUM | `abuse_link`, `abuse_other`, `feedback`, `other` |
| `subject` | TEXT | |
| `body` | TEXT | |
| `reported_url` | TEXT | |
| `short_id` | BIGINT NULL | resolve, если удалось |
| `client_ip` | INET | антиспам; **хешировать через 7 дней** → `client_ip_hash` |
| `consent_at` | TIMESTAMPTZ | момент согласия на обработку ПДн |
| `consent_text_version` | TEXT | версия текста согласия (например `support_guest_v1`) |
| `access_token_hash` | BYTEA | hash magic-link токена для guest follow-up |
| `guest_note` | TEXT NULL | одно уточнение гостя (≤2 KB) |
| `guest_note_at` | TIMESTAMPTZ NULL | время уточнения |
| `is_reviewed` | BOOLEAN DEFAULT FALSE | рассмотрено оператором |
| `reviewed_at` | TIMESTAMPTZ | |
| `admin_action` | ENUM NULL | `none`, `noted`, `short_deactivated`, `escalated` |
| `created_at` | TIMESTAMPTZ | |

Retention: **90 дней** после `is_reviewed=true`, иначе **180 дней** с
`created_at` → hard delete (sweeper).

### Guest follow-up (magic-link)

После `POST /support/reports` сервер:

1. Генерирует opaque token (32 bytes), хранит **только hash**.
2. Шлёт email: «Жалоба принята. Статус / уточнение: `{base}/support/r/{token}`».
3. `GET /support/reports/by-token` (token в path/query) — статус + возможность
   **одного** дополнительного сообщения в `guest_note` (макс 1–2 KB).

Без аккаунта, без полноценного чата — дёшево и достаточно для уточнений.

### `support_tickets`

| Колонка | Тип | Описание |
|---------|-----|----------|
| `id` | UUID PK | |
| `user_id` | UUID FK | |
| `subject` | TEXT | |
| `status` | ENUM | **`awaiting_admin` \| `awaiting_user` \| `closed`** |
| `priority_score` | INT | см. [очередь](#очередь-и-приоритеты) |
| `subscription_snapshot` | subscription_type | |
| `scope_id` | BIGINT NULL | опциональный контекст |
| `short_id` | BIGINT NULL | опциональный контекст |
| `closed_by` | ENUM NULL | `user`, `admin` |
| `closed_at` | TIMESTAMPTZ | |
| `reopen_until` | TIMESTAMPTZ | `closed_at + 7d` |
| `deleted_at` | TIMESTAMPTZ | soft (user); hard после retention |
| `created_at` / `updated_at` | | |

Нет статуса `open` — при создании сразу `awaiting_admin`.

### `support_messages`

| Колонка | Тип | Описание |
|---------|-----|----------|
| `id` | UUID | |
| `ticket_id` | UUID | |
| `author_type` | ENUM | `user`, `admin` |
| `body` | TEXT | max **8 KB** |
| `is_internal` | BOOLEAN DEFAULT FALSE | **internal notes** — только admin, не отдавать user API |
| `created_at` | | |

`author_admin_id` не нужен (один оператор).

### `support_attachments`

| Колонка | Тип | Описание |
|---------|-----|----------|
| `id` | UUID | |
| `message_id` | UUID | |
| `ticket_id` | UUID | денорм для ACL без JOIN |
| `storage_key` | TEXT | |
| `content_type` | TEXT | всегда `image/webp` после обработки |
| `size_bytes` | INT | |
| `width` / `height` | INT | |
| `sha256` | CHAR(64) | dedup: повторный upload → тот же key |

**Download:** `GET /support/attachments/{id}` — только владелец тикета или
service token. Иначе 404 (не 403 — меньше enumeration).

### `support_operator` (0..1 row)

| Колонка | Тип |
|---------|-----|
| `telegram_user_id` | BIGINT |
| `telegram_username` | TEXT NULL |
| `authenticated_at` | TIMESTAMPTZ |
| `last_seen_at` | TIMESTAMPTZ |

Минимум ПДн из Telegram: **id + username** (без first/last name в БД).

Модель inbox / campaigns / preferences — в
[`module_notification`](module_notification.md). Ответ admin в тикет создаёт
`user_notifications` (`kind=support_reply`).

### Простой audit (дёшево)

Таблица `support_audit_log` (append-only, retention **1 год**):

```text
at, action, entity_type, entity_id, meta jsonb
```

Примеры `action`: `ticket_close`, `report_review`, `short_deactivate`,
`campaign_create`. Без PII в `meta` (только uuid / short_id).

---

## Соответствие 152-ФЗ

> Это инженерный чеклист под [152-ФЗ](https://www.consultant.ru/document/cons_doc_LAW_61801/),
> не юридическое заключение. Политика конфиденциальности на сайте обязательна.

### Принципы (ст. 5)

| Принцип | Как в модуле |
|---------|----------------|
| Законность / цель | Цели: обработка обращений; модерация запрещённого контента; продуктовые уведомления (отдельное согласие на promo) |
| Минимизация | Не слать в TG **лишнее**: email пользователя и сырой IP — по запросу `/email`; в push сразу — текст обращения и фото (нужны для разбора) |
| Срок хранения | Явные TTL + sweeper (см. ниже) |
| Точность | User может править профиль; тикеты — append-only messages |
| Локализация (ст. 18 ч. 5) | **Первичная** БД и object storage в РФ. Telegram получает копии текста/фото для оператора (трансграничная передача) — зафиксировать в политике конфиденциальности |

### Согласие (ст. 9)

| Субъект | Основание |
|---------|-----------|
| Зарегистрированный user | Согласие при регистрации / оферта + отдельный toggle promo |
| Гость (`support_reports`) | Обязательный checkbox `consent=true` + `consent_text_version`; без него 400 |
| Promo-рассылки | Только `in_app_promo=true` (opt-in) |

Текст согласия гостя хранить версионированным в коде/CMS (`support_guest_v1`),
в БД — только version + timestamp.

### Права субъекта

| Право | Реализация |
|-------|------------|
| Доступ | Свои тикеты / notifications API; гость — magic-link |
| Удаление | `DELETE /support/tickets/{id}` + hard purge по TTL; удаление аккаунта каскадом |
| Отзыв согласия на promo | `PATCH /notifications/preferences` |
| Возражение | Отписка от promo; support нельзя «отписать» пока тикет открыт — закрыть и удалить |

### Что уходит в Telegram оператору

| В push / карточке тикета | Не в авто-push (только по команде) |
|--------------------------|-------------------------------------|
| `ticket_id`, status, subscription, priority | Полный email (`/email <uuid>`) |
| **Текст** subject + body сообщений | Сырой `client_ip` |
| **Фото** вложений (сжатый WebP с API) | Пароли / токены |

Источник истины — Postgres/object storage в РФ; Telegram — рабочая копия для разбора.

### Retention (уничтожение / обезличивание)

| Данные | TTL |
|--------|-----|
| Closed ticket + messages | **180 дней** после `closed_at` → hard delete |
| Soft-deleted ticket | **30 дней** → hard delete |
| Attachments | вместе с message; orphan GC ежедневно |
| `support_reports` reviewed | **90 дней** после review |
| `support_reports` open | **180 дней** с create |
| `user_notifications` | **30 дней** |
| `client_ip` plaintext | **7 дней** → заменить hash |
| `support_audit_log` | **365 дней** |
| Magic-link token | **14 дней** |

Sweeper: тот же процесс, что stats retention (или cron в http-api), batch DELETE.

### Иные меры

- Шифрование диска / managed Postgres в РФ (инфра).
- Access: service token + bot password; нет публичных admin UI в v1.
- Инцидент: процедура в operational runbook (вне этой спеки).

---

## Удешевление эксплуатации

| Решение | Экономия |
|---------|----------|
| WebP q=70, max сторона **1280** (не 1920) | −50–70% диск vs JPEG |
| Max **3** вложения / сообщение, **2 MB** raw upload | меньше CPU/сети |
| Dedup по sha256 + **content-addressed** ключ `support/{sha}.webp` | один объект на уникальный скрин |
| TTL + hard delete + **S3 lifecycle 180d** | диск не растёт годами |
| Private bucket, раздача только через API | нет public CDN / egress «на всех» |
| **Без versioning** на bucket | −×N места и API cost |
| Нет email-канала в v1 | нет SES/SMTP cost |
| Один bot process, webhook | нет long-polling workers |
| Internal notes без отдельных таблиц | `is_internal` flag |
| Guest notes ≤2 KB text | без вложений у гостя |

Object storage:

| Среда | Бэкенд |
|-------|--------|
| local (cargo без Docker) | `object_storage.backend: local` → `attachments_dir` |
| docker-compose.dev | **MinIO** (S3 API), bucket `support-attachments` |
| prod | тот же конфиг → **Yandex Object Storage / Selectel** (РФ) |

Lifecycle rule на bucket = удаление объектов старше **180 дней** (страховка к
sweeper в Postgres). Init: `infrastructure/minio/init-bucket.sh`.

---

## Очередь и приоритеты

```
priority_score = price_rub
               + min(age_hours, 72) * 2          -- старые не голодают
               + kind_bonus                      -- abuse отдельно
```

| Источник | kind_bonus |
|----------|------------|
| Обычный тикет | 0 |
| `support_reports` abuse_link | **10_000** |
| `support_reports` иное | 500 |

Сортировка: `priority_score DESC`, `created_at ASC`.

Пересчёт `age_bonus` — при выдаче очереди (вычисляемое поле), не UPDATE каждую
минуту.

---

## Abuse → действие по ссылке

При `PATCH …/reports/{id}` с `is_reviewed=true`:

```json
{
  "is_reviewed": true,
  "admin_action": "short_deactivated"
}
```

| `admin_action` | Эффект |
|----------------|--------|
| `noted` | только флаг |
| `short_deactivated` | если `short_id` известен → `ShortService` archive/deactivate + audit |
| `escalated` | пометить для ручного follow-up (SQL) |

Авто-deactivate **только** явно выбранным действием оператора (не молча).

---

## Вложения

| Правило | Значение |
|---------|----------|
| Upload | jpeg/png/webp/gif → **WebP q=70**, max side **1280** |
| Max raw | **2 MB** |
| Max count | **3** / message |
| Guest | вложения **запрещены** |
| Store | MinIO/S3 (`AttachmentStore`) или local FS |
| Key | content-addressed `support/{sha256}.webp` |
| ACL | download только owner ticket / service token |
| Virus | magic-bytes only в v1 (без тяжёлого AV) |

Конфиг (`support.object_storage`):

```yaml
object_storage:
  backend: s3          # local | s3
  endpoint: "http://minio:9000"
  region: "us-east-1"  # для MinIO любое; в Yandex — ru-central1
  bucket: "support-attachments"
  access_key: "…"
  secret_key: "…"
  force_path_style: true   # MinIO; у Yandex обычно false
  key_prefix: "support"
```

---

## Admin bot

См. [`support_bot.md`](../support_bot.md).

Кратко: 1 оператор, пароль, webhook;
в чат уходят **текст и фото**; email/IP — по явной команде; canned replies в конфиге.

---

## Лимиты

| Лимит | Значение |
|-------|----------|
| Guest reports / IP / ч | 5 |
| Guest reports / email / сут | 5 |
| Captcha guest | **обязательна** |
| Open tickets / user | 5 |
| Messages / ticket / ч | 20 |
| Reopen | Пока `now < closed_at + 7d` (TTL окна); **без** лимита на число reopen |
| Campaigns / сут | 5 |

---

## Сводная таблица эндпоинтов

| Метод | Путь | Auth | Описание |
|-------|------|------|----------|
| `POST` | `/support/reports` | — + captcha + consent | Гостевая жалоба |
| `GET` | `/support/reports/by-token` | magic token | Статус / 1 уточнение |
| `PATCH` | `/support/admin/reports/{id}` | service | Review + action |
| `POST` | `/support/tickets` | 🔒 | Создать |
| `GET` | `/support/tickets` | 🔒 | Список |
| `GET` | `/support/tickets/{id}` | 🔒 | Карточка |
| `POST` | `/support/tickets/{id}/messages` | 🔒 | Ответ user |
| `POST` | `/support/tickets/{id}/attachments` | 🔒 | Upload |
| `POST` | `/support/tickets/{id}/close` | 🔒 | Закрыть |
| `POST` | `/support/tickets/{id}/reopen` | 🔒 | Reopen в окне 7d (TTL) |
| `DELETE` | `/support/tickets/{id}` | 🔒 | Soft-delete |
| `GET` | `/support/admin/tickets` | service | Очередь |
| `POST` | `/support/admin/tickets/{id}/messages` | service | Ответ (+ internal) |
| `POST` | `/support/admin/tickets/{id}/close` | service | Закрыть |
| `DELETE` | `/support/admin/tickets/{id}` | service | Hard delete |
| `GET` | `/support/attachments/{id}` | 🔒 / service | ACL download |

Inbox / prefs / polling / campaigns — см. [`module_notification`](module_notification.md).
