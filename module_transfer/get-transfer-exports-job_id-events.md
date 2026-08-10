<a id="get-transfer-exports-job_id-events"></a>

### <span style="background:#7CB342;padding:5px">GET</span> `/transfer/{scope_id:int}/exports/{job_id:uuid}/events`

|                | Описание                                                                 |
|----------------|--------------------------------------------------------------------------|
| **Назначение** | SSE-стрим прогресса export job (основной способ ожидания completion)     |
| **Auth**       | Bearer или X-Api-Key (`transfer_export`)                                 |
| **Логика**     | 1. Проверяет доступ и что job — export в данном scope.                   |
|                | 2. Сразу шлёт `event: snapshot` с текущим `TransferJobResponse`.         |
|                | 3. Далее `progress`; терминальные `completed` / `failed` / `cancelled`   |
|                | &nbsp;&nbsp;&nbsp;закрывают stream.                                      |
| **Параметры**  | `scope_id`, `job_id` — path                                              |

---

| Kind                         | Код | Описание                         |
|------------------------------|-----|----------------------------------|
|                              | 200 | `text/event-stream`              |
| auth_error                   | 401 | Не авторизован                   |
| permission_denied_error      | 403 | Недостаточно прав API key        |
| api_key_scope_mismatch_error | 403 | API key / scope mismatch         |
| job_not_found_error          | 404 | Job не найден / не export         |
| server_error                 | 500 | Внутренняя ошибка                |

---

<details open>
<summary><b>Пример события</b></summary>

```text
event: snapshot
data: {"job_id":"550e8400-…","kind":"export","status":"running","progress":{"percent":26,…},…}

event: progress
data: {"job_id":"550e8400-…","status":"running","progress":{"percent":80,…},…}

event: completed
data: {"job_id":"550e8400-…","status":"completed","result":{"download_url":"…",…},…}
```

Заголовки ответа: `Content-Type: text/event-stream; charset=utf-8`,
`Cache-Control: no-cache`, `X-Accel-Buffering: no`.

</details>
