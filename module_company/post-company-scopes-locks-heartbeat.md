<a id="post-company-scopes-locks-heartbeat"></a>

### <span style="background:#42A5F5;padding:5px">POST</span> `/company/scopes/{scope_id:int}/locks/shorts/{short_id:int}/heartbeat`

|                | Описание                                                         |
|----------------|------------------------------------------------------------------|
| **Назначение** | Продлить TTL soft-lock (пока открыт редактор)                    |
| **Логика**     | 1. Bearer: текущий holder.                                       |
|                | 2. Нет lock / чужой holder → 409 `short_edit_locked_error`       |
|                | &nbsp;&nbsp;&nbsp;или 404 `lock_not_found`.                      |
|                | 3. EXPIRE / refresh value + новый `expires_at`.                  |
| **Параметры**  | `scope_id`, `short_id` — path                                    |
| **Auth**       | 🔒 Bearer holder                                                 |

---

| Kind                    | Код | Описание              |
|-------------------------|-----|-----------------------|
|                         | 200 | Продлено              |
| auth_error              | 401 | Не авторизован        |
| scope_forbidden_error   | 403 | Нет доступа           |
| lock_not_found          | 404 | Lock отсутствует      |
| short_edit_locked_error | 409 | Lock у другого user   |
| server_error            | 500 | Внутренняя ошибка     |

---

<details open>
<summary><b>Пример ответа 200</b></summary>

```json
{
  "short_id": 42,
  "expires_at": "2026-09-30T12:01:30Z"
}
```

</details>

> Клиент шлёт heartbeat примерно раз в `lock_ttl / 2` (≤ 20 с при TTL 45 с).
