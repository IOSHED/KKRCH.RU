<a id="post-payments"></a>

### <span style="background:#42A5F5;padding:5px">POST</span> `/payments`

|                | Описание                                                                                         |
|----------------|--------------------------------------------------------------------------------------------------|
| **Назначение** | Создать платёж в ЮKassa и получить `confirmation_token` для виджета                              |
| **Auth**       | Bearer                                                                                           |
| **Логика**     | 1. Валидирует `plan` (платный тариф). `FREE` / `FREE_PLUS` → 400 `plan_not_purchasable_error`.   |
|                | 2. Определяет сценарий (см. [модуль](../module_payment.md#покупка-продление-и-апгрейд)):         |
|                | &nbsp;&nbsp;&nbsp;- **purchase / renewal** — обязателен `months` ∈ [1, 12];                      |
|                | &nbsp;&nbsp;&nbsp;- **upgrade** — `plan` дороже текущего и `subscription_ends_at > now`;         |
|                | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`months` не передаётся;                                            |
|                | &nbsp;&nbsp;&nbsp;- **понижение** (`price_rub` меньше) при активном периоде → 409                |
|                | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`plan_downgrade_forbidden_error`.                                  |
|                | 3. Считает `amount_rub`: полная цена за `months` (−20% при 12) либо [формула апгрейда](../module_payment.md#формула-апгрейда). |
|                | 4. Создаёт `payments` (`pending`) + ЮKassa `POST /v3/payments`                                   |
|                | &nbsp;&nbsp;&nbsp;(`capture=true`, `confirmation.type=embedded`, metadata с `payment_id`).       |
|                | 5. Сохраняет `yookassa_payment_id`, `confirmation_token`; отвечает 201.                          |
|                | 6. Применение тарифа — только после webhook `payment.succeeded`.                                 |
| **Параметры**  | тело JSON (см. ниже)                                                                             |

> Виджет:
> [basics](https://yookassa.ru/developers/payment-acceptance/integration-scenarios/widget/basics).
> Успех виджета ≠ применение тарифа.

---

| Kind                            | Код | Описание                                      |
|---------------------------------|-----|-----------------------------------------------|
|                                 | 201 | Платёж создан, токен для виджета              |
| validation_error                | 400 | Невалидные поля                               |
| plan_not_purchasable_error      | 400 | `FREE` / `FREE_PLUS` / неизвестный план       |
| months_validation_error         | 400 | `months` вне 1..12 или лишний/пропущенный     |
| upgrade_amount_too_small_error  | 400 | Доплата апгрейда &lt; 1 ₽                     |
| auth_error                      | 401 | Нет или некорректный access token             |
| plan_downgrade_forbidden_error  | 409 | Целевой тариф дешевле текущего при активном периоде |
| payment_provider_error          | 502 | Ошибка / таймаут ЮKassa                       |
| server_error                    | 500 | Внутренняя ошибка                             |

---

<details open>
<summary><b>Пример — покупка / продление</b></summary>

```json
{
  "plan": "PERSONAL",
  "months": 12
}
```

</details>

<details open>
<summary><b>Пример — апгрейд (только plan)</b></summary>

```json
{
  "plan": "PRO"
}
```

</details>

<details open>
<summary><b>Пример ответа 201 — renewal</b></summary>

```json
{
  "payment_id": "550e8400-e29b-41d4-a716-446655440000",
  "yookassa_payment_id": "30b3e7d0-000f-5000-8000-1ed1588d6b0e",
  "kind": "renewal",
  "status": "pending",
  "plan": "PERSONAL",
  "from_plan": "PERSONAL",
  "months": 12,
  "days_granted": 360,
  "amount": {
    "value": "1910.00",
    "currency": "RUB"
  },
  "confirmation_token": "ct-30b3e7d0-000f-5000-8000-1ed1588d6b0e",
  "return_url": "https://ккрч.рф/billing/result",
  "created_at": "2026-09-04T08:00:00Z"
}
```

</details>

<details open>
<summary><b>Пример ответа 201 — upgrade</b></summary>

```json
{
  "payment_id": "660e8400-e29b-41d4-a716-446655440111",
  "yookassa_payment_id": "30b3e7d0-000f-5000-8000-1ed1588d6b1f",
  "kind": "upgrade",
  "status": "pending",
  "plan": "PRO",
  "from_plan": "PERSONAL",
  "months": null,
  "days_granted": 0,
  "remaining_days": 20,
  "amount": {
    "value": "133.00",
    "currency": "RUB"
  },
  "confirmation_token": "ct-30b3e7d0-000f-5000-8000-1ed1588d6b1f",
  "return_url": "https://ккрч.рф/billing/result",
  "created_at": "2026-09-04T12:00:00Z"
}
```

```text
floor( (399 − 199) × 20 / 30 ) = 133
```

</details>

<details>
<summary><b>Поля тела запроса</b></summary>

| Поле | Тип | Обязательность | Описание |
|------|-----|----------------|----------|
| `plan` | `PERSONAL` \| `PRO` \| `BUSINESS` \| `BUSINESS_PLUS` | да | Целевой тариф |
| `months` | int 1..12 | для purchase/renewal | Срок; 12 → скидка 20%. Для upgrade **не** передавать |

</details>
