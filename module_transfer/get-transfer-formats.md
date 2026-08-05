<a id="get-transfer-formats"></a>

### <span style="background:#7CB342;padding:5px">GET</span> `/transfer/formats`

|                | Описание                                                                                  |
|----------------|-------------------------------------------------------------------------------------------|
| **Назначение** | Справочник поддерживаемых форматов export/import, адаптеров и native-схем колонок         |
| **Логика**     | 1. Возвращает статический каталог (версионируется `catalog_version`).                     |
|                | 2. Описывает MIME, gzip-поведение, адаптеры Linkly/Bitly и native column sets.          |
|                | 3. Query `schema=` — детальная схема одного набора (`native_shorts`, `native_clicks`).    |

---

| Kind         | Код | Описание              |
|--------------|-----|-----------------------|
|              | 200 | Каталог форматов      |
| server_error | 500 | Внутренняя ошибка     |

---

<details open>
<summary><b>Пример ответа (сокращённо)</b></summary>

```json
{
  "catalog_version": 1,
  "export_formats": [
    { "id": "csv", "mime": "text/csv; charset=utf-8", "supports_gzip": true },
    { "id": "tsv", "mime": "text/tab-separated-values", "supports_gzip": true },
    { "id": "ndjson", "mime": "application/x-ndjson", "supports_gzip": true },
    { "id": "json", "mime": "application/json", "supports_gzip": true, "max_uncompressed_bytes": 10485760 }
  ],
  "export_bundles": ["shorts", "clicks", "full"],
  "import_adapters": [
    {
      "id": "native",
      "description": "CSV/TSV/NDJSON с column_mapping",
      "detect": ["schema_version", "short_name", "targets_json"]
    },
    {
      "id": "linkly_links",
      "description": "Linkly Links export (links.csv)",
      "detect": ["id", "name", "url", "slug", "full_url", "clicks_total"],
      "sample": "links.csv",
      "default_stats_mode": "agg_only"
    },
    {
      "id": "linkly_clicks_pivot",
      "description": "Linkly pivot clicks (clicks_pivot.csv)",
      "detect": ["pivot_blocks"],
      "sample": "clicks_pivot.csv",
      "default_stats_mode": "agg_only"
    },
    {
      "id": "bitly_links",
      "description": "Bitly Links page CSV export",
      "detect": ["Link", "Destination URL", "Engagements"],
      "default_stats_mode": "agg_only"
    }
  ],
  "preflight_errors": [
    "transfer_not_payed_error",
    "short_name_not_payed_error",
    "subdomain_not_payed_error",
    "stats_retention_error"
  ],
  "import_flags": [
    { "name": "ignore_retention_limit", "default": false, "description": "Raw вне retention → agg only" },
    { "name": "import_stats_mode", "values": ["auto", "agg_only", "raw_and_agg"] }
  ],
  "polling": {
    "recommended_interval_sec": 2,
    "max_interval_sec": 30,
    "sse_supported": true
  }
}
```

</details>

<details>
<summary><b>Query: schema=native_shorts</b></summary>

Фрагмент ответа — полный список колонок export `shorts`:

```json
{
  "schema": "native_shorts",
  "schema_version": 1,
  "columns": [
    { "name": "short_id", "type": "int64", "export": true, "import": false },
    { "name": "short_name", "type": "string", "export": true, "import": true, "required": true },
    { "name": "targets_json", "type": "json", "export": true, "import": true, "required": true },
    { "name": "agg_total_clicks", "type": "int64", "export": true, "import": "merge_agg" },
    { "name": "agg_clicks_by_day_json", "type": "json", "export": true, "import": "merge_agg" }
  ]
}
```

</details>
