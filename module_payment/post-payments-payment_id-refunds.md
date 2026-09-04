<a id="post-payments-payment_id-refunds"></a>

### <span style="background:#42A5F5;padding:5px">POST</span> `/payments/{payment_id}/refunds`

|                | Описание                                                                                      |
|----------------|-----------------------------------------------------------------------------------------------|
| **Назначение** | Полный (≤3 дня) или частичный (pro-rata за неиспользованные дни) возврат через ЮKassa         |
| **Auth**       | Bearer (владелец)                                                                             |
| **Логика**     | 1. Находит succeeded/partially_refunded платёж пользователя.                                  |
|                | 2. Если `now − paid_at ≤ full_refund_days` (default 3) → **full**: вся оставшаяся сумма.      |
|                | 3. Иначе → **partial**:                                      |
|                | &nbsp;&nbsp;&nbsp;`unused_days = days_granted − used_days − days_revoked`;                    |
|                | &nbsp;&nbsp;&nbsp;`amount = floor(amount_rub × unused_days / days_granted)` (с поправкой на   |
|                | &nbsp;&nbsp;&nbsp;уже возвращённое — не превысить остаток).                                   |
|                | 4. `amount < 1₽` → 400 `refund_amount_too_small_error`.                                       |
|                | 5. Создаёт `payment_refunds` + ЮKassa `POST /v3/refunds` (Idempotence-Key).                   |
|                | 6. При синхронном `succeeded` — сразу `apply_refund_days`; иначе ждёт webhook                 |
|                | &nbsp;&nbsp;&nbsp;`refund.succeeded`.                                                         |
|                | 7. Уменьшает `subscription_ends_at` на `days_revoked`; при `ends_at ≤ now` — FREE + cooling.  |
| **Параметры**  | `payment_id:uuid` — path; тело опционально                                                    |

См. [возвраты ЮKassa](https://yookassa.ru/developers/payment-acceptance/after-the-payment/refunds)
и [модуль: Возвраты](../module_payment.md#возвраты).

---

| Kind                         | Код | Описание                                           |
|------------------------------|-----|----------------------------------------------------|
|                              | 201 | Возврат создан (`pending` / уже `succeeded`)       |
| validation_error             | 400 | Невалидное тело                                    |
| refund_amount_too_small_error| 400 | Сумма &lt; 1 ₽                                     |
| refund_window_closed_error   | 400 | Нечего возвращать (дни исчерпаны)                  |
| auth_error                   | 401 | Нет или некорректный access token                  |
| payment_not_found_error      | 404 | Нет платежа / чужой                                |
| already_refunded_error       | 409 | Уже полный возврат                                 |
| refund_in_progress_error     | 409 | Уже есть pending refund                            |
| refund_not_supported_error   | 409 | Способ оплаты не поддерживает partial/full         |
| payment_not_refundable_error | 409 | Статус платежа не позволяет возврат                |
| payment_provider_error       | 502 | Ошибка ЮKassa                                      |
| server_error                 | 500 | Внутренняя ошибка                                  |

---

<details open>
<summary><b>Пример запроса</b></summary>

```json
{}
```

> Тело пустое: сервер сам выбирает `full` vs `partial` по окну 3 дней.
> Явный `mode` / `amount` в v1 **не** принимаются (защита от занижения дней).

</details>

<details open>
<summary><b>Пример ответа 201 — полный возврат</b></summary>

```json
{
  "refund_id": "7c9e6679-7425-40de-944b-e07fc1f90ae7",
  "payment_id": "550e8400-e29b-41d4-a716-446655440000",
  "kind": "full",
  "status": "succeeded",
  "amount": {
    "value": "199.00",
    "currency": "RUB"
  },
  "days_revoked": 30,
  "subscription_ends_at": null,
  "created_at": "2026-09-05T10:00:00Z"
}
```

</details>

<details open>
<summary><b>Пример — частичный (день 10 из 30, PERSONAL 199 ₽)</b></summary>

```json
{
  "refund_id": "8d0e7780-8536-51ef-a55c-f18gd2g01bf8",
  "payment_id": "550e8400-e29b-41d4-a716-446655440000",
  "kind": "partial",
  "status": "succeeded",
  "amount": {
    "value": "132.00",
    "currency": "RUB"
  },
  "days_revoked": 20,
  "subscription_ends_at": "2026-09-14T08:01:12Z",
  "created_at": "2026-09-14T08:01:12Z"
}
```

```text
used_days = 10
unused_days = 20
refund = floor(199 × 20 / 30) = 132 ₽
subscription_ends_at -= 20 days
```

</details>
