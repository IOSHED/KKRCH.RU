# Feedback Module

In-app опрос **полезности сервиса** после первых успешных кликов по ссылке.

> **Статус:** реализовано (`GET /feedback/status`, `POST /feedback`, `PATCH /feedback`).

Не путать с guest-жалобами `support_report_kind=feedback` в [`module_support`](module_support.md).

---

## Оглавление

| Иконка | Раздел | Ссылка |
|--------|--------|--------|
| 🎯 | Use cases | [ссылка](#use-cases) |
| 🗄️ | Модель данных | [ссылка](#модель-данных) |
| 🛡️ | Антиспам | [ссылка](#антиспам) |
| 📋 | Эндпоинты | [ссылка](#сводная-таблица-эндпоинтов) |

Endpoint-файлы: [`module_feedback/`](module_feedback/).

---

## Use cases

| Сценарий | Кто | Как |
|----------|-----|-----|
| Показать slide-in в кабинете | User | UI: создал ссылку → поделился → ≥3 клика; API: `GET /feedback/status` → `eligible` |
| Сохранить оценку (звёзды) | User | `POST /feedback` с `rating` 1..5 |
| Добавить комментарий | User | `PATCH /feedback` после оценки (опционально) |

Текст вопроса в UI: «Насколько полезен сервис для работы со ссылками?»  
(не «Оцените нас»).

---

## Модель данных

```text
user_feedback  →  1 row / user (PK = user_id)
```

| Поле | Тип | Описание |
|------|-----|----------|
| `user_id` | UUID PK | FK → `users.id` |
| `rating` | SMALLINT 1..5 | Основная метрика |
| `comment` | TEXT NULL | Бонус; ≤2000 символов |
| `created_at` | timestamptz | Момент оценки |
| `updated_at` | timestamptz | Момент комментария / правки |

Миграция: `20270101001400_user_feedback`.

**Eligibility:** у пользователя есть хотя бы одна ссылка с `shorts.clicks_count ≥ 3`
(lifetime human-клики).

---

## Антиспам

Оценка и комментарий привязаны к аккаунту: повторный `POST` → `409 feedback_already_submitted_error`.
Комментарий дописывается один раз (`409 comment_already_set_error`).

Dismiss «не показывать 30 дней» — только клиентский UX (localStorage), не API.

---

## Сводная таблица эндпоинтов

| Метод | Путь | Назначение |
|-------|------|------------|
| GET | [`/feedback/status`](module_feedback/get-feedback-status.md) | Eligibility + факт отправки |
| POST | [`/feedback`](module_feedback/post-feedback.md) | Сохранить оценку (+ опц. комментарий) |
| PATCH | [`/feedback`](module_feedback/patch-feedback.md) | Дописать комментарий к оценке |
