<a id="post-feedback"></a>

### <span style="background:#0288D1;padding:5px">POST</span> `/feedback`

|                | Описание |
|----------------|----------|
| **Назначение** | Сохранить оценку полезности (основная метрика продукта) |
| **Логика**     | 1. Bearer обязателен |
|                | 2. Проверка eligibility: есть ссылка с ≥ `min_clicks` кликов |
|                | 3. `INSERT` в `user_feedback`; PK `user_id` → один раз на аккаунт |
|                | 4. `comment` опционален; можно дописать позже через PATCH |
| **Параметры**  | Body JSON: |
|                | `rating:int` — 1..5 (обязательно) |
|                | `comment:string?` — ≤2000 символов |
| **Заголовки**  | `Authorization: Bearer <access_token>` |
|                | `Content-Type: application/json` |

---

| Kind                              | Код | Описание              |
|-----------------------------------|-----|-----------------------|
|                                   | 201 | Оценка сохранена      |
| validation_error                  | 400 | Невалидный rating/comment |
| not_eligible_error                | 403 | Ещё нет нужных кликов |
| feedback_already_submitted_error  | 409 | Оценка уже есть       |
| auth_error                        | 401 | Не авторизован        |
| server_error                      | 500 | Ошибка                |

---

<details open>
<summary><b>Пример запроса</b></summary>

```json
{
  "rating": 4,
  "comment": null
}
```

</details>

<details>
<summary><b>Пример ответа 201</b></summary>

```json
{
  "rating": 4,
  "comment": null,
  "submitted_at": "2026-08-10T18:00:00Z"
}
```

</details>
