<a id="get-transfer-imports-job_id"></a>

### <span style="background:#7CB342;padding:5px">GET</span> `/transfer/{scope_id:int}/imports/{job_id:uuid}`

|                | Описание                                                                                  |
|----------------|-------------------------------------------------------------------------------------------|
| **Назначение** | Polling статуса import job, прогресса и итоговой сводки                                   |
| **Логика**     | 1. Проверяет Bearer и ownership job / scope.                                                |
|                | 2. Возвращает `progress` (строки), счётчики created/skipped/failed.                         |
|                | 3. При `completed` — ссылки на `errors_download_url` если были ошибки строк.                |
|                | 4. При `dry_run=true` — только отчёт валидации без side effects.                            |
| **Параметры**  | `scope_id:int`, `job_id:uuid` — path                                                      |

---

| Kind                  | Код | Описание                         |
|-----------------------|-----|----------------------------------|
|                       | 200 | Статус import job                |
| auth_error            | 401 | Не авторизован                   |
| scope_not_found_error | 404 | Scope недоступен                 |
| job_not_found_error   | 404 | Job не найден                    |
| server_error          | 500 | Внутренняя ошибка                |

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
