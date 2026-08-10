<a id="patch-feedback"></a>

### <span style="background:#FB8C00;padding:5px">PATCH</span> `/feedback`

|                | Описание |
|----------------|----------|
| **Назначение** | Дописать текстовый комментарий к уже сохранённой оценке |
| **Логика**     | 1. Bearer обязателен |
|                | 2. Строка `user_feedback` должна существовать (сначала POST) |
|                | 3. Комментарий задаётся один раз (`comment IS NULL`) |
|                | 4. Пустой / whitespace → 400 |
| **Параметры**  | Body JSON: |
|                | `comment:string` — 1..2000 символов |
| **Заголовки**  | `Authorization: Bearer <access_token>` |
|                | `Content-Type: application/json` |

---

| Kind                      | Код | Описание                    |
|---------------------------|-----|-----------------------------|
|                           | 200 | Комментарий сохранён        |
| validation_error          | 400 | Пустой / слишком длинный    |
| feedback_not_found_error  | 404 | Оценка ещё не отправлена    |
| comment_already_set_error | 409 | Комментарий уже есть        |
| auth_error                | 401 | Не авторизован              |
| server_error              | 500 | Ошибка                      |

---

<details open>
<summary><b>Пример запроса</b></summary>

```json
{
  "comment": "Не хватает экспорта статистики по дням"
}
```

</details>
