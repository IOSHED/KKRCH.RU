<a id="post-payments-webhook"></a>

### <span style="background:#42A5F5;padding:5px">POST</span> `/payments/webhook`

|                | Описание                                                                               |
|----------------|----------------------------------------------------------------------------------------|
| **Назначение** | Приём [входящих уведомлений ЮKassa](https://yookassa.ru/developers/using-api/webhooks) |
| **Логика**     | 1. Парсит тело `{ event, object }`.                                                    |
|                | 2. Идемпотентность: `event_id` / hash → таблица `payment_webhook_events`; дубль → 200. |
|                | 3. Для `payment.*` — GET `/v3/payments/{id}` у ЮKassa; сверить `status`, `amount`,     |
|                | &nbsp;&nbsp;&nbsp;`metadata.payment_id`. Расхождение → 400, без мутаций.               |
|                | 4. `payment.succeeded` → по `kind`: purchase/renewal начисляют дни; upgrade меняет     |
|                | &nbsp;&nbsp;&nbsp;`subscription` без сдвига `ends_at`; сброс cooling.                  |
|                | 5. `payment.canceled` → статус платежа `canceled`.                                     |
|                | 6. `refund.succeeded` → `apply_refund_days` (если ещё не применено синхронно).         |
|                | 7. `refund.canceled` → статус refund `canceled` + `cancellation_details`.              |
|                | 8. Всегда отвечает **200** на успешно принятое (в т.ч. unknown event — log + ignore).  |
| **Параметры**  | тело JSON от ЮKassa                                                                    |

> Клиентский success виджета **не** заменяет webhook. Без успешного webhook
> тариф не меняется.

---

| Kind                     | Код | Описание                                 |
|--------------------------|-----|------------------------------------------|
|                          | 200 | Принято (в т.ч. идемпотентный повтор)    |
| webhook_validation_error | 400 | Невалидное тело / сверка с API провалена |
| webhook_forbidden_error  | 403 | IP не в allowlist (если включён)         |
| server_error             | 500 | Внутренняя ошибка (ЮKassa ретраит)       |

---

<details open>
<summary><b>Пример тела — payment.succeeded</b></summary>

```json
{
  "type": "notification",
  "event": "payment.succeeded",
  "object": {
    "id": "30b3e7d0-000f-5000-8000-1ed1588d6b0e",
    "status": "succeeded",
    "amount": {
      "value": "1910.00",
      "currency": "RUB"
    },
    "metadata": {
      "payment_id": "550e8400-e29b-41d4-a716-446655440000",
      "user_id": "11111111-2222-3333-4444-555555555555",
      "plan": "PERSONAL",
      "months": "12"
    },
    "paid": true,
    "created_at": "2026-09-04T08:00:05.000Z"
  }
}
```

</details>

<details>
<summary><b>Обрабатываемые события</b></summary>

| Event               | Действие                               |
|---------------------|----------------------------------------|
| `payment.succeeded` | purchase/renewal — дни; upgrade — plan, ends_at без сдвига |
| `payment.canceled`  | `payments.status = canceled`           |
| `refund.succeeded`  | Списать дни / обновить refunded_amount |
| `refund.canceled`   | Пометить refund canceled               |
| прочие              | 200 + log                              |

</details>
