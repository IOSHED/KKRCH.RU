<a id="get-transfer-exports-job_id"></a>

### <span style="background:#7CB342;padding:5px">GET</span> `/transfer/{scope_id:int}/exports/{job_id:uuid}`

|                | Описание                                                                                  |
|----------------|-------------------------------------------------------------------------------------------|
| **Назначение** | Polling статуса export job и прогресса длительной выгрузки                                |
| **Логика**     | 1. Проверяет Bearer и что job принадлежит `scope_id`.                                     |
|                | 2. Читает job из Postgres (`transfer_jobs`).                                              |
|                | 3. Возвращает `status`, `progress`, `result` (если completed) или `error`.                |
|                | 4. Клиент поллит каждые **2–30 с** до terminal status.                                    |
| **Параметры**  | `scope_id:int`, `job_id:uuid` — path                                                      |

> **SSE не поддерживается в v1.** Не используйте `Accept: text/event-stream`.

---

| Kind                  | Код | Описание                         |
|-----------------------|-----|----------------------------------|
|                       | 200 | Статус job                       |
| auth_error            | 401 | Не авторизован                   |
| scope_not_found_error | 404 | Scope недоступен                 |
| job_not_found_error   | 404 | Job не найден                    |
| server_error          | 500 | Внутренняя ошибка                |

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

<details open>
<summary><b>Пример — completed</b></summary>

```json
{
  "job_id": "550e8400-e29b-41d4-a716-446655440000",
  "kind": "export",
  "status": "completed",
  "progress": {
    "phase": "done",
    "rows_processed": 4808420,
    "rows_total": 4808420,
    "bytes_written": 18432003,
    "percent": 100
  },
  "created_at": "2026-07-21T09:00:00Z",
  "started_at": "2026-07-21T09:00:02Z",
  "finished_at": "2026-07-21T09:04:18Z",
  "expires_at": "2026-07-22T09:00:00Z",
  "error": null,
  "result": {
    "download_url": "/api/v1/transfer/10000001/exports/550e8400-e29b-41d4-a716-446655440000/download",
    "filename": "full-scope-10000001-20260721.zip",
    "size_bytes": 18432003,
    "sha256": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
    "row_counts": { "shorts": 8420, "clicks": 4800000 }
  }
}
```

</details>

<details>
<summary><b>Пример — failed</b></summary>

```json
{
  "job_id": "550e8400-e29b-41d4-a716-446655440000",
  "kind": "export",
  "status": "failed",
  "error": {
    "kind": "export_too_large_error",
    "reason": "Оценка выгрузки 6.2 GB превышает лимит тарифа PRO (5 GB)"
  }
}
```

</details>
