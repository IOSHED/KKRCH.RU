<a id="get-payments"></a>

### <span style="background:#7CB342;padding:5px">GET</span> `/payments`

|                | Описание                                                              |
|----------------|-----------------------------------------------------------------------|
| **Назначение** | История платежей текущего пользователя (новые сверху)                 |
| **Auth**       | Bearer                                                                |
| **Логика**     | 1. Валидирует access token.                                           |
|                | 2. Читает `payments` пользователя с пагинацией.                       |
|                | 3. `confirmation_token` **не** отдаётся (только на create).           |
| **Параметры**  | `limit:int?` (default 20, max 100); `offset:int?` (default 0)         |
|                | `status:string?` — фильтр по статусу                                  |

---

| Kind         | Код | Описание                          |
|--------------|-----|-----------------------------------|
|              | 200 | Список платежей                   |
| auth_error   | 401 | Нет или некорректный access token |
| server_error | 500 | Внутренняя ошибка сервера         |

---

<details open>
<summary><b>Пример ответа 200</b></summary>

```json
{
  "items": [
    {
      "payment_id": "550e8400-e29b-41d4-a716-446655440000",
      "status": "succeeded",
      "kind": "renewal",
      "plan": "PERSONAL",
      "from_plan": "PERSONAL",
      "months": 12,
      "days_granted": 360,
      "days_revoked": 0,
      "amount": {
        "value": "1910.00",
        "currency": "RUB"
      },
      "refunded_amount": {
        "value": "0.00",
        "currency": "RUB"
      },
      "paid_at": "2026-09-04T08:01:12Z",
      "coverage_start_at": "2026-09-04T08:01:12Z",
      "created_at": "2026-09-04T08:00:00Z"
    }
  ],
  "total": 1,
  "limit": 20,
  "offset": 0
}
```

</details>
