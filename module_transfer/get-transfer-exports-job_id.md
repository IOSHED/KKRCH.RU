<a id="get-transfer-exports-job_id"></a>

### <span style="background:#7CB342;padding:5px">GET</span> `/transfer/{scope_id:int}/exports/{job_id:uuid}`

|                | Описание                                                                                  |
|----------------|-------------------------------------------------------------------------------------------|
| **Назначение** | Одноразовый snapshot статуса export job (не polling-loop)                                 |
| **Логика**     | 1. Проверяет Bearer/API key и что job принадлежит `scope_id`.                             |
|                | 2. Читает job из Postgres (`transfer_jobs`).                                              |
|                | 3. Возвращает `status`, `progress`, `result` / `error`.                                   |
| **Прогресс**   | Основной канал — **SSE** `GET …/exports/{job_id}/events` (`events_url` из 202).            |
| **Параметры**  | `scope_id:int`, `job_id:uuid` — path                                                      |

---

| Kind                         | Код | Описание                         |
|------------------------------|-----|----------------------------------|
|                              | 200 | Статус job                       |
| auth_error                   | 401 | Не авторизован                   |
| permission_denied_error      | 403 | Недостаточно прав API key        |
| api_key_scope_mismatch_error | 403 | API key / scope mismatch         |
| scope_not_found_error        | 404 | Scope недоступен                 |
| job_not_found_error          | 404 | Job не найден / не export         |
| server_error                 | 500 | Внутренняя ошибка                |

---

<details open>
<summary><b>Пример — running</b></summary>

```json
{
  "job_id": "550e8400-e29b-41d4-a716-446655440000",
  "kind": "export",
  "status": "running",
  "progress": {
    "phase": "writing_clicks",
    "rows_processed": 1250000,
    "rows_total": 4800000,
    "bytes_written": 943718400,
    "percent": 26
  },
  "created_at": "2026-07-21T09:00:00Z",
  "started_at": "2026-07-21T09:00:02Z",
  "finished_at": null,
  "expires_at": "2026-07-22T09:00:00Z",
  "error": null,
  "result": null
}
```

</details>
