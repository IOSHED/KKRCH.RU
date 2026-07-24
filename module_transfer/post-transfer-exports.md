<a id="post-transfer-exports"></a>

### <span style="background:#42A5F5;padding:5px">POST</span> `/transfer/{scope_id:int}/exports`

|                | Описание                                                                                  |
|----------------|-------------------------------------------------------------------------------------------|
| **Назначение** | Асинхронный export ссылок и/или raw-статистики scope в файл (CSV / TSV / NDJSON / ZIP)    |
| **Логика**     | 1. Проверяет Bearer и доступ к `scope_id`.                                                  |
|                | 2. Проверяет `subscription_plans.transfer_enabled` (FREE → 402).                          |
|                | 3. Валидирует тело: `bundle`, `format`, фильтры, окно кликов.                               |
|                | 3. Если уже есть active export job для scope → 409 `export_job_conflict_error`.             |
|                | 4. Создаёт job (`status=pending`), ставит в очередь `transfer-worker`.                      |
|                | 5. Worker stream-записывает файл(ы) с gzip; прогресс обновляет Postgres.                  |
|                | 6. Возвращает `202 Accepted` + `job_id` и URL для polling.                                  |
| **Параметры**  | `scope_id:int` — path                                                                     |

---

| Kind                        | Код | Описание                                      |
|-----------------------------|-----|-----------------------------------------------|
|                             | 202 | Export job создан                             |
| validation_error            | 400 | Некорректные фильтры / окно / format          |
| auth_error                  | 401 | Не авторизован                                |
| scope_not_found_error       | 404 | Scope недоступен                              |
| export_job_conflict_error   | 409 | Уже выполняется export в этом scope           |
| export_too_large_error      | 413 | Оценка размера превышает лимит подписки       |
| transfer_not_payed_error    | 402 | Модуль недоступен на FREE                     |
| too_many_requests_error     | 429 | Rate limit transfer                           |
| server_error                | 500 | Внутренняя ошибка                             |

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
  "download_url": "/api/v1/transfer/10000001/exports/550e8400-e29b-41d4-a716-446655440000/download",
  "expires_at": "2026-07-22T09:00:00Z",
  "estimated_rows": {
    "shorts": 8420,
    "clicks": 4800000
  }
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
| `tags_mode` | `any` \| `all` | `any` | |
| `clicks.from` / `clicks.to` | datetime | retention window | Только для `clicks` / `full` |
| `clicks.include_bots` | bool | `false` | Raw export ботов |

</details>
