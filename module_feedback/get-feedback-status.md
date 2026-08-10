<a id="get-feedback-status"></a>

### <span style="background:#7CB342;padding:5px">GET</span> `/feedback/status`

|                | Описание |
|----------------|----------|
| **Назначение** | Статус опроса: можно ли показать UI и сохранена ли уже оценка |
| **Логика**     | 1. Bearer обязателен |
|                | 2. Читает `user_feedback` для текущего пользователя |
|                | 3. Считает `clicks_total` = `SUM(shorts.clicks_count)` по scopes пользователя |
|                | 4. `eligible = !submitted && EXISTS(short с clicks_count ≥ min_clicks)` |
| **Параметры**  | — |
| **Заголовки**  | `Authorization: Bearer <access_token>` |

---

| Kind         | Код | Описание       |
|--------------|-----|----------------|
|              | 200 | Статус         |
| auth_error   | 401 | Не авторизован |
| server_error | 500 | Ошибка         |

---

<details open>
<summary><b>Пример ответа</b></summary>

```json
{
  "eligible": true,
  "submitted": false,
  "clicks_total": 12,
  "min_clicks": 3,
  "rating": null,
  "has_comment": false,
  "submitted_at": null
}
```

</details>
