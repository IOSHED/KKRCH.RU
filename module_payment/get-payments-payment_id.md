<a id="get-payments-payment_id"></a>

### <span style="background:#7CB342;padding:5px">GET</span> `/payments/{payment_id}`

|                | Описание                                                                 |
|----------------|--------------------------------------------------------------------------|
| **Назначение** | Статус одного платежа (поллинг после виджета / экран billing)            |
| **Auth**       | Bearer (только владелец)                                                 |
| **Логика**     | 1. Валидирует access token.                                              |
|                | 2. Ищет платёж по `id` + `user_id`; иначе 404.                           |
|                | 3. Опционально (если `pending` старше N сек) — сверка с ЮKassa GET       |
|                | &nbsp;&nbsp;&nbsp;`/v3/payments/{yookassa_payment_id}` (не чаще 1/15с).  |
|                | 4. Возвращает снимок + краткий preview доступного возврата.              |
| **Параметры**  | `payment_id:uuid` — path                                                 |

> Применение тарифа всё равно идёт через webhook; сверка — только для UX
> «оплата прошла, ждём webhook».

---

| Kind                 | Код | Описание                          |
|----------------------|-----|-----------------------------------|
|                      | 200 | Снимок платежа                    |
| auth_error           | 401 | Нет или некорректный access token |
| payment_not_found_error | 404 | Нет платежа / чужой              |
| server_error         | 500 | Внутренняя ошибка сервера         |

---

<details open>
<summary><b>Пример ответа 200</b></summary>

```json
{
  "payment_id": "550e8400-e29b-41d4-a716-446655440000",
  "yookassa_payment_id": "30b3e7d0-000f-5000-8000-1ed1588d6b0e",
  "kind": "purchase",
  "status": "succeeded",
  "plan": "PERSONAL",
  "from_plan": null,
  "months": 1,
  "days_granted": 30,
  "days_revoked": 0,
  "amount": {
    "value": "199.00",
    "currency": "RUB"
  },
  "refunded_amount": {
    "value": "0.00",
    "currency": "RUB"
  },
  "paid_at": "2026-09-04T08:01:12Z",
  "coverage_start_at": "2026-09-04T08:01:12Z",
  "created_at": "2026-09-04T08:00:00Z",
  "refund_preview": {
    "eligible": true,
    "mode": "full",
    "full_refund_until": "2026-09-07T08:01:12Z",
    "unused_days": 30,
    "estimated_amount": {
      "value": "199.00",
      "currency": "RUB"
    }
  }
}
```

</details>

<details>
<summary><b>Поля `refund_preview`</b></summary>

| Поле | Описание |
|------|----------|
| `eligible` | Можно ли сейчас запросить возврат |
| `mode` | `full` \| `partial` \| `none` |
| `full_refund_until` | Граница 3-дневного полного возврата |
| `unused_days` | Оценка неиспользованных дней покрытия |
| `estimated_amount` | Оценка суммы к возврату (до вызова ЮKassa) |

</details>
