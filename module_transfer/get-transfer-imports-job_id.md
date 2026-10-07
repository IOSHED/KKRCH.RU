<a id="get-transfer-imports-job_id"></a>

### <span style="background:#7CB342;padding:5px">GET</span> `/transfer/{scope_id:int}/imports/{job_id:uuid}`

|                | Описание                                                                                                                                            |
|----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------|
| **Назначение** | Одноразовый snapshot статуса import job (не polling-loop)                                                                                           |
| **Auth**       | Bearer или X-Api-Key (`transfer_import`)                                                                                                            |
| **Логика**     | 1. Job должен быть `kind=import` в данном scope.                                                                                                    |
|                | 2. Возвращает `TransferJobResponse` (progress, result / error).                                                                                     |
|                | 3. При `completed` — `result.download_url` (CSV созданных ссылок, если они есть), `result.errors_download_url` / `sample_errors` при ошибках строк. |
| **Прогресс**   | Основной канал — **SSE** [`…/events`](get-transfer-imports-job_id-events.md).                                                                       |
| **Параметры**  | `scope_id:int`, `job_id:uuid` — path                                                                                                                |

---

| Kind                         | Код | Описание                  |
|------------------------------|-----|---------------------------|
|                              | 200 | Статус import job         |
| auth_error                   | 401 | Не авторизован            |
| permission_denied_error      | 403 | Недостаточно прав API key |
| api_key_scope_mismatch_error | 403 | API key / scope mismatch  |
| job_not_found_error          | 404 | Job не найден / не import |
| server_error                 | 500 | Внутренняя ошибка         |

---

<details open>
<summary><b>Пример — running</b></summary>

```json
{
  "job_id": "660e8400-e29b-41d4-a716-446655440001",
  "kind": "import",
  "status": "running",
  "adapter": "linkly_links",
  "progress": {
    "phase": "inserting_shorts",
    "rows_processed": 4200,
    "rows_total": 8420,
    "percent": 50
  },
  "created_at": "2026-07-21T10:00:00Z",
  "started_at": "2026-07-21T10:00:01Z",
  "finished_at": null,
  "error": null,
  "result": null
}
```

</details>

<details open>
<summary><b>Пример — completed (partial)</b></summary>

```json
{
  "job_id": "660e8400-e29b-41d4-a716-446655440001",
  "kind": "import",
  "status": "completed",
  "adapter": "linkly_links",
  "progress": {
    "phase": "done",
    "rows_processed": 8420,
    "rows_total": 8420,
    "percent": 100
  },
  "finished_at": "2026-07-21T10:03:44Z",
  "error": null,
  "result": {
    "created_shorts": 8312,
    "updated_agg": 842000,
    "raw_events_inserted": 158000,
    "raw_events_skipped_retention": 842000,
    "skipped": 18,
    "failed": 90,
    "warnings_count": 124,
    "errors_download_url": "/api/v1/transfer/10000001/imports/660e8400-e29b-41d4-a716-446655440001/errors",
    "sample_errors": [
      {
        "row": 42,
        "kind": "short_name_conflict_error",
        "reason": "slug 2ni5v уже занят в subdomain"
      }
    ]
  }
}
```

</details>

<details>
<summary><b>Пример — dry_run completed</b></summary>

```json
{
  "status": "completed",
  "result": {
    "dry_run": true,
    "would_create": 8312,
    "would_fail": 90,
    "validation_report_url": "/api/v1/transfer/10000001/imports/660e8400…/errors"
  }
}
```

</details>
