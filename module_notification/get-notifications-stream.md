<a id="get-notifications-stream"></a>

### <span style="background:#7CB342;padding:5px">GET</span> `/notifications/stream`

|                | Описание                                                                      |
|----------------|-------------------------------------------------------------------------------|
| **Назначение** | SSE near-instant (opt-in). Default клиент — polling.                          |
| **Логика**     | 1. Только `?ticket=` из [`POST …/stream-ticket`](post-notifications-stream-ticket.md). |
|                | 2. **Запрещён** `access_token` в query.                                       |
|                | 3. Отдаёт одноразовый `event: snapshot` с текущим inbox и **закрывает** соединение (не long-lived Redis pub/sub). Near-instant — polling на клиенте. |

---

| Kind                  | Код | Описание        |
|-----------------------|-----|-----------------|
|                       | 200 | SSE             |
| ticket_invalid_error  | 401 | Ticket истёк    |
| server_error          | 500 | Ошибка          |

---
