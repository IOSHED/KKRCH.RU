<a id="post-transfer-exports"></a>

### <span style="background:#42A5F5;padding:5px">POST</span> `/transfer/{scope_id:int}/exports`

|                | Описание                                                                                  |
|----------------|-------------------------------------------------------------------------------------------|
| **Назначение** | Асинхронный export ссылок и/или raw-статистики scope в файл (CSV / TSV / NDJSON / ZIP)    |
| **Auth**       | Bearer или X-Api-Key (`transfer_export`)                                                  |
| **Логика**     | 1. Gate тарифа: `transfer_daily_bytes > 0` → иначе 402 `transfer_not_payed_error`.         |
|                | 2. Валидирует тело: `bundle`, `format`, `compress`, фильтры, окно кликов.                   |
|                | 3. Активный export на scope → 409 `export_job_conflict_error`.                             |
|                | 4. Суточная квота байт → 429 `too_many_requests_error`.                                    |
|                | 5. Создаёт job (`pending`), поднимает SSE-канал, `tokio::spawn` runner в `http-api`.       |
|                | 6. Ответ 202: `poll_url`, **`events_url`**, `download_url`, `expires_at`.                   |
| **Параметры**  | `scope_id:int` — path                                                                     |

> Прогресс — через SSE (`events_url`). Snapshot GET — после reconnect.

---

| Kind                         | Код | Описание                                      |
|------------------------------|-----|-----------------------------------------------|
|                              | 202 | Export job создан                             |
| validation_error             | 400 | Некорректные фильтры / окно / format          |
| auth_error                   | 401 | Не авторизован                                |
| permission_denied_error      | 403 | Недостаточно прав API key                     |
| api_key_scope_mismatch_error | 403 | API key привязан к другому scope              |
| scope_not_found_error        | 404 | Scope недоступен                              |
| export_job_conflict_error    | 409 | Уже выполняется export в этом scope           |
| transfer_not_payed_error     | 402 | `transfer_daily_bytes = 0` на тарифе          |
| too_many_requests_error      | 429 | Исчерпан суточный объём переноса              |
| server_error                 | 500 | Внутренняя ошибка                             |

---

<details open>
<summary><b>Пример запроса — shorts + agg, CSV gzip</b></summary>

```json
{
  "bundle": "shorts",
  "format": "csv",
  "compress": "gzip",
  "include_archived": true,
  "include_bots_in_agg": false,
  "folder_id": null,
  "tags": [],
  "tags_mode": "any"
}
```

</details>

<details open>
<summary><b>Пример запроса — full backup</b></summary>

```json
{
  "bundle": "full",
  "format": "csv",
  "compress": "gzip",
  "clicks": {
    "from": "2026-01-01T00:00:00Z",
    "to": "2026-07-21T23:59:59Z",
    "include_bots": false
  }
}
```

</details>

<details open>
<summary><b>Пример ответа 202</b></summary>

```json
{
  "job_id": "550e8400-e29b-41d4-a716-446655440000",
  "kind": "export",
  "status": "pending",
  "poll_url": "/api/v1/transfer/10000001/exports/550e8400-e29b-41d4-a716-446655440000",
  "events_url": "/api/v1/transfer/10000001/exports/550e8400-e29b-41d4-a716-446655440000/events",
  "download_url": "/api/v1/transfer/10000001/exports/550e8400-e29b-41d4-a716-446655440000/download",
  "expires_at": "2026-07-22T09:00:00Z"
}
```

</details>

<details>
<summary><b>Поля тела запроса</b></summary>

| Поле | Тип | Default | Описание |
|------|-----|---------|----------|
| `bundle` | `shorts` \| `clicks` \| `full` | **обяз.** | Что выгружать |
| `format` | `csv` \| `tsv` \| `ndjson` \| `json` | `csv` | Формат строк |
| `compress` | `gzip` \| `none` | `gzip` | Сжатие результата |
| `include_archived` | bool | `true` | Включать archived shorts |
| `include_bots_in_agg` | bool | `false` | Добавить bot-колонки agg |
| `folder_id` | uuid? | null | Фильтр по папке |
| `tags` | string[] | `[]` | Фильтр по тегам |
| `tags_mode` | `any` \| `all` | null | Режим тегов |
| `clicks` | object? | null | Окно raw-кликов для `clicks` / `full` |
| `columns` | string[]? | null | Подмножество колонок shorts (порядок сохраняется) |
| `click_columns` | string[]? | null | Для `full`: колонки кликов отдельно |

</details>
