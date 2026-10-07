<a id="post-transfer-imports"></a>

### <span style="background:#42A5F5;padding:5px">POST</span> `/transfer/{scope_id:int}/imports`

|                | Описание                                                                                         |
|----------------|--------------------------------------------------------------------------------------------------|
| **Назначение** | Асинхронный import ссылок/статистики из CSV / TSV / NDJSON                                       |
| **Auth**       | Bearer или X-Api-Key (`transfer_import`)                                                         |
| **Content-Type** | `application/json` (не multipart)                                                              |
| **Логика**     | 1. Gate тарифа: `transfer_daily_bytes > 0` → иначе 402 `transfer_not_payed_error`.                |
|                | 2. Активный import на scope → 409 `import_job_conflict_error`.                                   |
|                | 3. Источник данных: `inline_csv` и/или `upload_relative_path`.                                   |
|                | 4. `inline_csv` > `transfer.max_import_upload_bytes` → 400 `validation_error`.                   |
|                | 5. При `inline_csv` пишет артефакт `uploads/{job_id}.csv`, создаёт job, `tokio::spawn` runner. |
|                | 6. Ответ 202: `poll_url`, **`events_url`** (SSE), `detected_adapter`, `expires_at`.              |
|                | 7. Лимиты shorts/subdomain/retention на этой ручке **не** гейтятся — см. [preflight](post-transfer-imports-preflight.md). |
| **Параметры**  | `scope_id:int` — path                                                                            |

> Клиент следит за прогрессом через SSE (`events_url`). Snapshot GET — после reconnect, не как polling-loop.

---

| Kind                        | Код | Описание                                              |
|-----------------------------|-----|-------------------------------------------------------|
|                             | 202 | Import job создан                                     |
| validation_error            | 400 | metadata / размер inline / формат                     |
| auth_error                  | 401 | Не авторизован                                        |
| permission_denied_error     | 403 | Недостаточно прав API key                             |
| api_key_scope_mismatch_error| 403 | API key привязан к другому scope                      |
| scope_not_found_error       | 404 | Scope недоступен                                      |
| import_job_conflict_error   | 409 | Уже выполняется import                                |
| transfer_not_payed_error    | 402 | `transfer_daily_bytes = 0` на тарифе                  |
| too_many_requests_error     | 429 | Исчерпан суточный объём переноса (байты)              |
| server_error                | 500 | Внутренняя ошибка                                     |

---

<details open>
<summary><b>Пример запроса — native CSV inline</b></summary>

```json
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
  "import_stats_mode": "raw_and_agg",
  "on_row_error": "continue",
  "dry_run": false,
  "inline_csv": "Slug,Title,Destinations\nhello,Landing,\"[{\\\"url\\\":\\\"https://example.com\\\",\\\"weight\\\":100}]\"\n"
}
```

</details>

<details open>
<summary><b>Пример запроса — Linkly links</b></summary>

```json
{
  "adapter": "linkly_links",
  "format": "csv",
  "match_subdomain": true,
  "default_redirect_type": 302,
  "import_stats_mode": "agg_only",
  "on_row_error": "continue",
  "inline_csv": "<содержимое links.csv>"
}
```

`clicks_total` из Linkly → merge в `link_short_agg.total_clicks` (без raw).

</details>

<details open>
<summary><b>Пример запроса — Bitly</b></summary>

```json
{
  "adapter": "bitly_links",
  "format": "csv",
  "import_stats_mode": "agg_only",
  "on_row_error": "continue",
  "skip_deleted": true,
  "inline_csv": "<CSV Bitly Links export>"
}
```

</details>

<details open>
<summary><b>Пример запроса — Linkly clicks pivot</b></summary>

```json
{
  "adapter": "linkly_clicks_pivot",
  "format": "csv",
  "match_by": "full_url",
  "import_stats_mode": "agg_only",
  "on_row_error": "continue",
  "inline_csv": "<clicks_pivot.csv>"
}
```

</details>

<details open>
<summary><b>Ответ 202</b></summary>

```json
{
  "job_id": "660e8400-e29b-41d4-a716-446655440001",
  "kind": "import",
  "status": "pending",
  "poll_url": "/api/v1/transfer/10000001/imports/660e8400-e29b-41d4-a716-446655440001",
  "events_url": "/api/v1/transfer/10000001/imports/660e8400-e29b-41d4-a716-446655440001/events",
  "detected_adapter": "native",
  "expires_at": "2026-07-22T10:00:00Z"
}
```

`detected_adapter` сейчас = переданный `adapter` (авто-детект по содержимому файла пока не выполняется на create).

</details>

<details>
<summary><b>Поля тела запроса</b></summary>

| Поле | Тип | Default | Описание |
|------|-----|---------|----------|
| `adapter` | string | `native` | `native`, `linkly_links`, `linkly_clicks_pivot`, `bitly_links` |
| `format` | string | `csv` | `csv`, `tsv`, `ndjson` |
| `column_mapping` | object? | null | CSV header → поле (для `native`) |
| `create_folders` | bool | `false` | Создавать папки из `folder_path` |
| `ignore_retention_limit` | bool | `false` | Raw вне retention → skip raw, write agg |
| `import_stats_mode` | string? | null/`auto` | `auto`, `agg_only`, `raw_and_agg` |
| `on_row_error` | string | `continue` | `continue` \| `abort` |
| `dry_run` | bool | `false` | Валидация без записи |
| `inline_csv` | string? | null | Тело файла (UTF-8); пишется в storage |
| `upload_relative_path` | string? | null | Уже загруженный объект в storage |
| `match_subdomain` | bool? | null | Linkly: матч subdomain |
| `default_redirect_type` | u16? | null | Default redirect (302/…) |
| `match_by` | string? | null | Pivot: `full_url`, `external_id`, `short_name` |
| `skip_deleted` | bool? | null | Bitly: пропускать deleted |
| `locale` | string? | null | Язык текстов ошибок строк: `ru`, `en`, `de`, `fr` |
| `defaults` | object? | null | Подставляется в пустые ячейки строки |

`defaults`: `subdomain` или `custom_domain` (один хост), `folder_id` или `folder_name` (папка ищется по имени без учёта регистра, иначе создаётся), `redirect_type`, `tags`, `description`, `is_captcha`, `is_active`, `max_clicks`. Явное значение в CSV не перезаписывается.

</details>
