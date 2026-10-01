<a id="get-company-scopes-events"></a>

### <span style="background:#7CB342;padding:5px">GET</span> `/company/scopes/{scope_id:int}/events`

|                | Описание                                                                 |
|----------------|--------------------------------------------------------------------------|
| **Назначение** | SSE-стрим совместной работы в explorer (shorts/folders + locks)          |
| **Auth**       | Bearer: owner или member с `shorts:read` **или** `folders:read`          |
| **Логика**     | 1. Проверяет доступ к scope.                                             |
|                | 2. Сразу `event: hello` + snapshot активных soft-lock.                   |
|                | 3. Подписка Redis Pub/Sub `company:collab:{scope_id}`.                   |
|                | 4. Периодический `event: ping` (`company.sse_heartbeat`).                |
|                | 5. События: `short_changed`, `folder_changed`, `short_lock`,             |
|                | &nbsp;&nbsp;&nbsp;`short_unlock`. История не хранится.                   |
| **Параметры**  | `scope_id` — path                                                        |

---

| Kind                       | Код | Описание                              |
|----------------------------|-----|---------------------------------------|
|                            | 200 | `text/event-stream`                   |
| auth_error                 | 401 | Не авторизован                        |
| permission_denied_error    | 403 | Member без read short/folders         |
| scope_forbidden_error      | 403 | Нет доступа к проекту                 |
| sse_limit_error            | 429 | Слишком много SSE у пользователя      |
| server_error               | 500 | Внутренняя ошибка                     |

---

<details open>
<summary><b>Пример событий</b></summary>

```text
event: hello
data: {"scope_id":10000001,"locks":[]}

event: short_changed
data: {"action":"updated","short_ids":[42],"actor":{"user_id":"…","display_name":"Иван"},"at":"2026-09-30T12:00:01Z"}

event: folder_changed
data: {"action":"created","folder_ids":["550e8400-…"],"actor":{"user_id":"…","display_name":"Иван"},"at":"…"}

event: short_lock
data: {"short_id":42,"user_id":"…","display_name":"Аня","expires_at":"2026-09-30T12:00:45Z"}

event: ping
data: {"t":"2026-09-30T12:00:15Z"}
```

Заголовки: `Content-Type: text/event-stream; charset=utf-8`,
`Cache-Control: no-cache`, `X-Accel-Buffering: no`.

</details>

> Подключать SSE только пока открыт explorer shared/company scope.
> Personal scope без команды — коннект допустим, но publish no-op.
