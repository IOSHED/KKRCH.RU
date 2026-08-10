<a id="get-transfer-formats"></a>

### <span style="background:#7CB342;padding:5px">GET</span> `/transfer/formats`

|                | Описание                                                                                  |
|----------------|-------------------------------------------------------------------------------------------|
| **Назначение** | Справочник форматов export/import, адаптеров и native-колонок                             |
| **Auth**       | Нет                                                                                       |
| **Логика**     | 1. Статический каталог (`catalog_version`).                                               |
|                | 2. Включает `export_formats`, `export_bundles`, `import_adapters`,                        |
|                | &nbsp;&nbsp;&nbsp;`native_shorts_columns`, `native_clicks_columns`, `import_flags`.       |
|                | 3. `polling.sse_supported=true`, `events_path_suffix=/events`; интервалы polling = null.  |
|                | 4. Query-параметров нет.                                                                  |

---

| Kind | Код | Описание         |
|------|-----|------------------|
|      | 200 | Каталог форматов |

---

<details open>
<summary><b>Пример ответа (сокращённо)</b></summary>

```json
{
  "catalog_version": 1,
  "export_formats": [
    { "id": "csv", "mime": "text/csv; charset=utf-8", "supports_gzip": true },
    { "id": "ndjson", "mime": "application/x-ndjson", "supports_gzip": true }
  ],
  "export_bundles": ["shorts", "clicks", "full"],
  "import_adapters": [
    { "id": "native", "detect": ["short_name", "long_url"] },
    { "id": "linkly_links", "detect": ["slug", "url", "name", "domain"] },
    { "id": "bitly_links", "detect": ["long_url", "title", "Destination URL"] }
  ],
  "polling": {
    "recommended_interval_sec": null,
    "max_interval_sec": null,
    "sse_supported": true,
    "events_path_suffix": "/events"
  }
}
```

</details>
