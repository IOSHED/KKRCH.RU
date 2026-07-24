<a id="post-transfer-imports"></a>

### <span style="background:#42A5F5;padding:5px">POST</span> `/transfer/{scope_id:int}/imports`

|                | Описание                                                                                  |
|----------------|-------------------------------------------------------------------------------------------|
| **Назначение** | Асинхронный import ссылок и/или статистики из CSV / TSV / NDJSON / gzip                    |
| **Логика**     | 0. **Gate:** `transfer_enabled` → 402 `transfer_not_payed_error` (FREE).                  |
|                | 1. **Preflight** (inline, см. [preflight](post-transfer-imports-preflight.md)):           |
|                | &nbsp;&nbsp;&nbsp;- лимит `max_shorts` → 402 `short_name_not_payed_error` + `over_limit_by`; |
|                | &nbsp;&nbsp;&nbsp;- отсутствующие subdomain → 402 `subdomain_not_payed_error` + `missing_subdomains[]`; |
|                | &nbsp;&nbsp;&nbsp;- raw вне retention → 400 `stats_retention_error` (если `ignore_retention_limit=false`). |
|                | 1. Проверяет Bearer и доступ к scope.                                                     |
|                | 2. Принимает `multipart/form-data` (file + JSON metadata).                                |
|                | 3. Auto-detect адаптера; `column_mapping` для `native`.                                    |
|                | 4. Создаёт import job; worker пишет shorts + **agg** (+ raw в retention).                   |
|                | 5. Строки с ошибками → `errors.ndjson.gz` (при `on_row_error=continue`).                  |
|                | 6. Возвращает `202 Accepted` + `job_id`.                                                  |
| **Параметры**  | `scope_id:int` — path                                                                     |

---

| Kind                        | Код | Описание                                              |
|-----------------------------|-----|-------------------------------------------------------|
|                             | 202 | Import job создан (preflight пройден)                   |
| stats_retention_error       | 400 | Raw-клики вне retention                               |
| validation_error            | 400 | metadata / mapping / формат                           |
| auth_error                  | 401 | Не авторизован                                        |
| scope_not_found_error       | 404 | Scope недоступен                                      |
| import_job_conflict_error   | 409 | Уже выполняется import                                |
| payload_too_large_error     | 413 | Файл > лимита upload                                  |
| short_name_not_payed_error  | 402 | `current + would_create > max_shorts`                 |
| subdomain_not_payed_error   | 402 | Subdomain из файла не созданы у пользователя          |
| transfer_not_payed_error    | 402 | Модуль недоступен на FREE (`transfer_enabled=false`)  |
| too_many_requests_error     | 429 | Rate limit                                            |
| server_error                | 500 | Внутренняя ошибка                                     |

---

<details open>
<summary><b>Metadata — флаги лимитов и статистики</b></summary>

```json
{
  "adapter": "native",
  "format": "csv",
  "ignore_retention_limit": false,
  "import_stats_mode": "raw_and_agg",
  "create_folders": true,
  "on_row_error": "continue",
  "dry_run": false
}
```

| Поле | Default | Описание |
|------|---------|----------|
| `ignore_retention_limit` | `false` | `true` — события старше retention **не** пишутся в raw, но **обновляют agg** |
| `import_stats_mode` | `auto` | `agg_only` \| `raw_and_agg` \| `auto` — см. [Import статистики](../module_transfer.md#import-статистики-raw--agg) |
| `dry_run` | `false` | Preflight + валидация строк без записи |

</details>

<details open>
<summary><b>Multipart — native CSV с mapping</b></summary>

```http
POST /api/v1/transfer/10000001/imports
Authorization: Bearer <access_token>
Content-Type: multipart/form-data; boundary=----boundary

------boundary
Content-Disposition: form-data; name="metadata"
Content-Type: application/json

{
  "adapter": "native",
  "format": "csv",
  "column_mapping": {
    "short_name": "Slug",
    "description": "Title",
    "targets_json": "Destinations"
  },
  "create_folders": true,
  "ignore_retention_limit": false,
  "on_row_error": "continue",
  "dry_run": false
}
------boundary
Content-Disposition: form-data; name="file"; filename="links.csv.gz"
Content-Type: application/gzip
Content-Encoding: gzip

<binary gzip stream>
------boundary--
```

</details>

<details open>
<summary><b>Metadata — Linkly links.csv (auto adapter)</b></summary>

```json
{
  "adapter": "linkly_links",
  "format": "csv",
  "match_subdomain": true,
  "default_redirect_type": 302,
  "import_stats_mode": "agg_only",
  "on_row_error": "continue"
}
```

`clicks_total` из Linkly → merge в `link_short_agg.total_clicks` (без raw).

Файл — [`links.csv`](../links.csv).

</details>

<details open>
<summary><b>Metadata — Bitly Links export</b></summary>

```json
{
  "adapter": "bitly_links",
  "format": "csv",
  "import_stats_mode": "agg_only",
  "on_row_error": "continue",
  "skip_deleted": true
}
```

`Engagements` → merge `total_clicks` в agg.

</details>

<details open>
<summary><b>Metadata — Linkly clicks pivot</b></summary>

```json
{
  "adapter": "linkly_clicks_pivot",
  "format": "csv",
  "match_by": "full_url",
  "import_stats_mode": "agg_only",
  "on_row_error": "continue"
}
```

Файл — [`clicks_pivot.csv`](../clicks_pivot.csv). Обновляет `clicks_by_day` и
связанные agg-счётчики.

</details>

<details open>
<summary><b>Metadata — native full re-import (shorts + raw clicks)</b></summary>

```json
{
  "adapter": "native",
  "bundle": "full",
  "format": "ndjson",
  "ignore_retention_limit": true,
  "import_stats_mode": "raw_and_agg"
}
```

ZIP `full` export: shorts → INSERT; clicks в retention → raw + agg;
clicks вне retention → **agg only** (при `ignore_retention_limit=true`).

</details>

<details open>
<summary><b>Ответ 202</b></summary>

```json
{
  "job_id": "660e8400-e29b-41d4-a716-446655440001",
  "kind": "import",
  "status": "pending",
  "poll_url": "/api/v1/transfer/10000001/imports/660e8400-e29b-41d4-a716-446655440001",
  "detected_adapter": "linkly_links",
  "preflight": {
    "would_create_shorts": 8312,
    "raw_events_within_retention": 158000,
    "raw_events_agg_only": 842000
  },
  "expires_at": "2026-07-22T10:00:00Z"
}
```

</details>

<details>
<summary><b>Поля metadata (полный список)</b></summary>

| Поле | Тип | Описание |
|------|-----|----------|
| `adapter` | enum | `native`, `linkly_links`, `linkly_clicks_pivot`, `bitly_links`, `auto` |
| `format` | enum | `csv`, `tsv`, `ndjson` |
| `bundle` | enum | `shorts`, `clicks`, `full` — для native multi-file / zip |
| `column_mapping` | object | CSV header → поле (только `native`) |
| `create_folders` | bool | Создавать `folder_path` |
| `ignore_retention_limit` | bool | Raw вне retention → skip raw, **write agg** |
| `import_stats_mode` | enum | `auto`, `agg_only`, `raw_and_agg` |
| `dry_run` | bool | Без записи |
| `on_row_error` | `continue` \| `abort` | Default `continue` |
| `match_by` | string | pivot: `full_url`, `external_id`, `short_name` |

</details>
