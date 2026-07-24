<a id="post-notifications-stream-ticket"></a>

### <span style="background:#42A5F5;padding:5px">POST</span> `/notifications/stream-ticket`

|                | Описание                                                                      |
|----------------|-------------------------------------------------------------------------------|
| **Назначение** | Выдать одноразовый ticket для SSE **без** Bearer в query (логи прокси)        |
| **Логика**     | 1. Bearer. 2. Redis `stream_ticket:{token}` → user_id, TTL **120 с**.          |
|                | 3. Клиент: `EventSource(/notifications/stream?ticket=…)`.                     |

---

| Kind         | Код | Описание       |
|--------------|-----|----------------|
|              | 200 | Ticket выдан   |
| auth_error   | 401 | Не авторизован |
| server_error | 500 | Ошибка         |

---

<details open>
<summary><b>Ответ</b></summary>

```json
{
  "ticket": "opaque-random",
  "expires_in": 120,
  "stream_url": "/api/v1/notifications/stream?ticket=opaque-random"
}
```

</details>

> Default UX — polling. SSE — opt-in для снижения числа соединений на тарифе.
