# Notification Module

Единая система уведомлений: **in-app inbox**, **browser** (клиентский
Notification API по inbox) и **email** (канал доставки через
[`mail_worker`](../mail_worker.md)).

Тикеты / жалобы / admin bot — в [`module_support`](module_support.md).
Ответ оператора создаёт `user_notifications` (`kind=support_reply`) и при
включённом email — отложенную задачу в Redis.

---

## Оглавление

| Иконка | Раздел | Ссылка |
|--------|--------|--------|
| 🎯 | Use cases | [ссылка](#use-cases) |
| 🗄️ | Модель данных | [ссылка](#модель-данных) |
| 🔔 | Каналы | [ссылка](#каналы) |
| ⚡ | Polling / browser | [ссылка](#polling--browser) |
| 📧 | Email (отложенные и lifecycle) | [ссылка](#email-отложенные-и-lifecycle) |
| 📋 | Эндпоинты | [ссылка](#сводная-таблица-эндпоинтов) |

Endpoint-файлы: [`module_notification/`](module_notification/).

---

## Use cases

| Сценарий | In-app | Browser | Email |
|----------|--------|---------|-------|
| Ответ поддержки | сразу (если `in_app_support`) | сразу при появлении в inbox (если `browser_support`) | через `support_reply_email_delay`, если не прочитано / тикет не закрыт пользователем и всё ещё `awaiting_user` |
| Анонс / promo | campaign fan-out | как inbox | нет (только in-app opt-in) |
| Скоро кончится подписка | `subscription_ending` | как inbox | если `email_billing` |
| Скоро кончится cooling | `cooling_ending` | как inbox | если `email_billing` |
| Дайджест кликов 7d (churn) | нет | нет | если `email_digest`, не заходил ≥ `digest_inactive_after`, кликов за период **> 5** |

---

## Модель данных

```text
notification_campaigns  →  job fan-out (batch)
user_notifications      →  inbox (короткий TTL)
notification_preferences →  1 row / user
users.last_seen_at       →  активность для digest
```

Email-доставки **не** пишутся в отдельную PG-таблицу: очередь Redis
(`mail:queue` / `mail:delayed`) + идемпотентность `SET NX EX`.

| `notification_preferences` | default |
|----------------------------|---------|
| `in_app_support` | true |
| `in_app_release` | true |
| `in_app_promo` | **false** (opt-in) |
| `in_app_billing` | true |
| `browser_support` | true |
| `email_support` | **true** |
| `email_billing` | true |
| `email_digest` | true |

| `user_notifications` retention | **14 дней** (`support.notification_retention`) → DELETE на `GET /notifications` + глобальный sweep |

| `notification_kind` | |
|---------------------|--|
| `support_reply` | ответ поддержки |
| `release` / `promo` / `custom` | campaigns |
| `system` | служебные |
| `subscription_ending` | предупреждение об окончании подписки |
| `cooling_ending` | предупреждение об окончании cooling |
| `stats_digest` | только для email-задач (inbox не создаётся) |

Fan-out campaigns: батчи по **500** user_id; UNIQUE `(campaign_id, user_id)
WHERE campaign_id IS NOT NULL`. Точечный `support_reply` — plain `INSERT`.

---

## Каналы

| Канал | Поведение |
|--------|-----------|
| In-app inbox | `user_notifications` + polling ≥15 с |
| Browser | клиент: `Notification` API при новых unread и `browser_support`; Web Push (закрытая вкладка) — не в этом релизе |
| Email | SMTP Timeweb (`info@…`) через `mail-worker` |
| Telegram admin | [Support Bot](../support_bot.md) |

Prefs: `GET/PATCH /notifications/preferences`.

---

## Polling / browser

1. Admin reply (`is_internal=false`) → при `in_app_support` — строка inbox
   (`kind=support_reply`, `payload.ticket_id`).
2. Клиент: `GET /notifications` ≥ **15 с**.
3. При росте unread и разрешении браузера + `browser_support` — системное
   уведомление ОС (без отдельного backend push).

SSE / stream-ticket **не** поддерживаются.

---

## Email (отложенные и lifecycle)

| Событие | Когда ставится в очередь | Условие отправки |
|---------|--------------------------|------------------|
| Support reply | сразу после admin reply (`ZADD mail:delayed`, delay из конфига) | `email_support`; inbox `!is_read`; тикет `awaiting_user` и не закрыт |
| Subscription ending | schedule в `mail-worker` | `ends_at` в окне warn; `email_billing` / `in_app_billing` |
| Cooling ending | schedule в `mail-worker` | `cooling_until` в окне warn |
| Stats digest | schedule | `last_seen_at` старше порога; есть клики за 7d; `email_digest` |

Локально: `mail.file.override_to` перенаправляет все письма на тестовый адрес;
`mail-worker --send-test` шлёт одно тестовое письмо.

Детали очереди, SMTP и метрик — [`mail_worker.md`](../mail_worker.md).

---

## Сводная таблица эндпоинтов

| Метод | Путь | Auth | Описание |
|-------|------|------|----------|
| `GET` | `/notifications` | 🔒 | Inbox |
| `PATCH` | `/notifications/{id}/read` | 🔒 | Read |
| `GET`/`PATCH` | `/notifications/preferences` | 🔒 | Prefs / каналы |
| `POST` | `/support/admin/notifications` | service | Campaign |
