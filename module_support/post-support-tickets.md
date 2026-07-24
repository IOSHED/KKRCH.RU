<a id="post-support-tickets"></a>

### <span style="background:#42A5F5;padding:5px">POST</span> `/support/tickets`

|                | Описание                                                              |
|----------------|-----------------------------------------------------------------------|
| **Назначение** | Создать обращение (сразу `awaiting_admin`)                            |
| **Логика**     | 1. Bearer; лимит открытых тикетов (5).                                |
|                | 2. `priority_score` из price_rub (+ age later at read).               |
|                | 3. Optional `scope_id` / `short_id` — контекст (ownership check).     |
|                | 4. INSERT ticket + first message; событие боту с **текстом** и URL фото.  |

---

| Kind                        | Код | Описание              |
|-----------------------------|-----|-----------------------|
|                             | 201 | Создан                |
| validation_error            | 400 | subject / body        |
| auth_error                  | 401 | Не авторизован        |
| too_many_open_tickets_error | 409 | Лимит открытых        |
| server_error                | 500 | Ошибка                |

---

<details open>
<summary><b>Пример запроса</b></summary>

```json
{
  "subject": "Не работает редирект с паролем",
  "body": "После ввода пароля снова форма.",
  "short_id": 10000042
}
```

</details>
