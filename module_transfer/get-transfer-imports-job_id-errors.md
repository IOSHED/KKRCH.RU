<a id="get-transfer-imports-job_id-errors"></a>

### <span style="background:#7CB342;padding:5px">GET</span> `/transfer/{scope_id:int}/imports/{job_id:uuid}/errors`

|                | Описание                                                                                  |
|----------------|-------------------------------------------------------------------------------------------|
| **Назначение** | Скачивание NDJSON-отчёта об ошибках и предупреждениях import (gzip)                       |
| **Логика**     | 1. Доступно после `status=completed` или `failed` (если успел накопить errors).            |
|                | 2. Каждая строка — `{ "row", "kind", "reason", "raw_preview"? }`.                         |
|                | 3. Stream gzip; формат фиксирован `application/x-ndjson`.                                   |
| **Параметры**  | `scope_id:int`, `job_id:uuid` — path                                                      |

---

| Kind                    | Код | Описание                         |
|-------------------------|-----|----------------------------------|
|                         | 200 | Файл errors.ndjson.gz            |
| auth_error              | 401 | Не авторизован                   |
| job_not_found_error     | 404 | Job не найден                    |
| errors_not_found_error  | 404 | Ошибок не было / файл expired    |
| server_error            | 500 | Внутренняя ошибка                |

---

<details open>
<summary><b>Пример строк NDJSON (после распаковки gzip)</b></summary>

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
