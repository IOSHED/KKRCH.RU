<a id="get-transfer-imports-job_id-events"></a>

### <span style="background:#7CB342;padding:5px">GET</span> `/transfer/{scope_id:int}/imports/{job_id:uuid}/events`

|                | Описание                                                                 |
|----------------|--------------------------------------------------------------------------|
| **Назначение** | SSE-стрим прогресса import job                                           |
| **Auth**       | Bearer или X-Api-Key (`transfer_import`)                                 |
| **Логика**     | Аналогично [export events](get-transfer-exports-job_id-events.md):       |
|                | snapshot → progress → terminal; URL из `events_url` ответа 202 create.   |
| **Параметры**  | `scope_id`, `job_id` — path                                              |

---

| Kind                         | Код | Описание                         |
|------------------------------|-----|----------------------------------|
|                              | 200 | `text/event-stream`              |
| auth_error                   | 401 | Не авторизован                   |
| permission_denied_error      | 403 | Недостаточно прав API key        |
| api_key_scope_mismatch_error | 403 | API key / scope mismatch         |
| job_not_found_error          | 404 | Job не найден / не import         |
| server_error                 | 500 | Внутренняя ошибка                |
