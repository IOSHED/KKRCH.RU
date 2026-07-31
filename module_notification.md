# Notification Module

Модуль **in-app inbox**, preferences и admin campaigns.

Тикеты / жалобы / admin bot — в [`module_support`](module_support.md).
Ответ оператора создаёт `user_notifications` с `kind=support_reply`.

> **Статус:** реализовано (inbox, preferences, polling ≥15 с, campaigns).

---

## Оглавление

| Иконка | Раздел | Ссылка |
|--------|--------|--------|
| 🎯 | Use cases | [ссылка](#use-cases) |
| 🗄️ | Модель данных | [ссылка](#модель-данных) |
| 🔔 | Каналы | [ссылка](#каналы) |
| ⚡ | Polling | [ссылка](#polling) |
| 📋 | Эндпоинты | [ссылка](#сводная-таблица-эндпоинтов) |

Endpoint-файлы: [`module_notification/`](module_notification/).

---

## Use cases

| Сценарий | Кто | Как |
|----------|-----|-----|
| Ответ поддержки в UI | User | inbox + polling ≥15 с |
| Анонс релиза / promo | Admin | `POST /support/admin/notifications` (campaign) |
| Opt-out promo | User | `PATCH /notifications/preferences` |

---

## Модель данных

```text
notification_campaigns  →  job fan-out (batch)
user_notifications      →  inbox (короткий TTL)
notification_preferences →  1 row / user
```

**Без** таблицы `notification_deliveries` в v1 (email later — отдельная
миграция).

| `notification_preferences` | default |
|----------------------------|---------|
| `in_app_support` | true |
| `in_app_release` | true |
| `in_app_promo` | **false** (opt-in) |
| `email_support` | false (future) |

| `user_notifications` retention | **30 дней** или после `is_read` + 7 дней → DELETE |

| `notification_campaigns` | |
|--------------------------|--|
| `kind` | `support_reply` не в campaigns — только точечные inbox |
| `title`, `body`, `payload` | |
| `audience` | `all`, `paid`, `subscription_in` |
| `status` | `queued` → `sending` → `sent` |
| `cursor_user_id` | прогресс fan-out |
| `created_at` | |

Fan-out: батчи по **500** user_id, без длинной транзакции; идемпотентность —
частичный UNIQUE `(campaign_id, user_id) WHERE campaign_id IS NOT NULL`
(не `NULLS NOT DISTINCT`: иначе `support_reply` с `campaign_id IS NULL`
блокировал бы все ответы кроме первого).

Точечный `support_reply` из `admin_reply` — обычный `INSERT` без
`ON CONFLICT` (каждый ответ оператора → отдельная строка inbox).

Promo-рассылки — только при `in_app_promo=true` (opt-in). Отзыв:
`PATCH /notifications/preferences`.

---

## Каналы

| Канал | v1 |
|--------|-----|
| In-app inbox | да |
| Polling | да (**≥15 с**) |
| SSE | нет |
| Telegram user | нет |
| Email user | нет |
| Telegram admin | да — в [Support Bot](../support_bot.md) (текст + фото тикета) |

`POST /support/admin/notifications` → campaign job.

Prefs: `GET/PATCH /notifications/preferences`.

---

## Polling

1. Admin reply (`POST /support/admin/tickets/{id}/messages`, `is_internal=false`)
   → при `in_app_support` создаётся `user_notifications` (`kind=support_reply`,
   `payload.ticket_id`). Internal note уведомление **не** создаёт.
2. Клиент: `GET /notifications` с интервалом **≥ 15 с** (панель открыта и фон).
   Параметр `?since=` опционален.

SSE / stream-ticket в v1 **не** поддерживаются.

---

## Сводная таблица эндпоинтов

| Метод | Путь | Auth | Описание |
|-------|------|------|----------|
| `GET` | `/notifications` | 🔒 | Inbox |
| `PATCH` | `/notifications/{id}/read` | 🔒 | Read |
| `GET`/`PATCH` | `/notifications/preferences` | 🔒 | Prefs / opt-out promo |
| `POST` | `/support/admin/notifications` | service | Campaign |
