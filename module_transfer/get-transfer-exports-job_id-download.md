<a id="get-transfer-exports-job_id-download"></a>

### <span style="background:#7CB342;padding:5px">GET</span> `/transfer/{scope_id:int}/exports/{job_id:uuid}/download`

|                | Описание                                                                                  |
|----------------|-------------------------------------------------------------------------------------------|
| **Назначение** | Скачивание готового файла export (потоково, с gzip)                                       |
| **Логика**     | 1. Job должен быть в `status=completed` и не `expired`.                                   |
|                | 2. Проверяет Bearer и ownership scope.                                                    |
|                | 3. Stream-отдаёт объект из storage; для `compress=gzip` — тело уже сжато.                 |
|                | 4. Поддерживает HTTP Range **не обязательно в v1** (целиком).                             |
| **Параметры**  | `scope_id:int`, `job_id:uuid` — path; `compress=gzip\|none` — query (default gzip)        |
| **Заголовки**  | `Accept: text/csv, application/x-ndjson, application/zip, */*`                            |

---

| Kind                    | Код | Описание                              |
|-------------------------|-----|---------------------------------------|
|                         | 200 | Файл (stream)                         |
| auth_error              | 401 | Не авторизован                        |
| job_not_found_error     | 404 | Job не найден                         |
| export_not_ready_error  | 409 | Job ещё не completed                  |
| export_expired_error    | 410 | TTL download истёк, файл удалён     |
| server_error            | 500 | Внутренняя ошибка                     |

---

<details open>
<summary><b>Пример запроса</b></summary>

```http
GET /api/v1/transfer/10000001/exports/550e8400-e29b-41d4-a716-446655440000/download?compress=gzip
Authorization: Bearer <access_token>
Accept: text/csv
```

</details>

<details open>
<summary><b>Пример ответа (заголовки)</b></summary>

```http
HTTP/1.1 200 OK
Content-Type: text/csv; charset=utf-8
Content-Encoding: gzip
Content-Disposition: attachment; filename*=UTF-8''shorts-scope-10000001-20260721.csv.gz
Content-Length: 18432003
X-Content-SHA256: e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
Cache-Control: private, no-store
```

Тело — бинарный gzip-поток. Браузер сохраняет файл; для programmatic download
используйте `response.blob()` и проверяйте `X-Content-SHA256`.

</details>

<details>
<summary><b>bundle=full → ZIP</b></summary>

```http
Content-Type: application/zip
Content-Disposition: attachment; filename*=UTF-8''full-scope-10000001-20260721.zip
```

Внутри ZIP файлы **уже** `.gz` (без повторного сжатия на уровне HTTP).

</details>
