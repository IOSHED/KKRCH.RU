<a id="get-transfer-imports-job_id-errors"></a>

### <span style="background:#7CB342;padding:5px">GET</span> `/transfer/{scope_id:int}/imports/{job_id:uuid}/errors`

|                | Описание                                                                                  |
|----------------|-------------------------------------------------------------------------------------------|
| **Назначение** | Скачивание отчёта по строкам, которые import не создал |
| **Логика**     | 1. Доступно, если у job есть файл ошибок. |
|                | 2. CSV: колонки `line`, `reason`, `row` (`row` — исходная строка файла). |
|                | 3. Старый sidecar остаётся gzip NDJSON (`application/x-ndjson`). |
| **Параметры**  | `scope_id:int`, `job_id:uuid` — path |

---

| Kind                    | Код | Описание                         |
|-------------------------|-----|----------------------------------|
|                         | 200 | CSV ошибок или gzip NDJSON   |
| auth_error              | 401 | Не авторизован                   |
| job_not_found_error     | 404 | Job не найден                    |
| errors_not_found_error  | 404 | Ошибок не было / файл expired    |
| server_error            | 500 | Внутренняя ошибка                |

---

<details open>
<summary><b>Пример CSV</b></summary>

```csv
line,reason,row
2,long_url указывает на домен сервиса,https://short.example/already
8,Короткое имя «promo» уже есть в проекте — строка пропущена,promo,https://example.com/promo
```

</details>

<details>
<summary><b>Заголовки ответа (CSV)</b></summary>

```http
HTTP/1.1 200 OK
Content-Type: text/csv; charset=utf-8
Content-Disposition: attachment; filename="import-errors-660e8400-e29b-41d4-a716-446655440001.csv"
```

</details>

<details>
<summary><b>Старый gzip NDJSON</b></summary>

```json
{"row":42,"kind":"short_name_conflict_error","reason":"slug 2ni5v уже занят","raw_preview":"41400247,Первая ссылка..."}
{"row":108,"kind":"long_url_validation_error","reason":"url не прошёл валидацию","field":"targets[0].url"}
{"row":512,"kind":"import_warning","reason":"Linkly column fb_pixel_id не поддерживается — пропущено"}
```

</details>

<details open>
<summary><b>Заголовки ответа</b></summary>

```http
HTTP/1.1 200 OK
Content-Type: application/x-ndjson
Content-Encoding: gzip
Content-Disposition: attachment; filename*=UTF-8''import-errors-660e8400….ndjson.gz
```

</details>
