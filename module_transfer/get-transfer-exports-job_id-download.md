<a id="get-transfer-exports-job_id-download"></a>

### <span style="background:#7CB342;padding:5px">GET</span> `/transfer/{scope_id:int}/exports/{job_id:uuid}/download`

|                | Описание                                                                                  |
|----------------|-------------------------------------------------------------------------------------------|
| **Назначение** | Скачивание готового файла export                                                          |
| **Auth**       | Bearer или X-Api-Key (`transfer_export`)                                                  |
| **Логика**     | 1. Job `kind=export`, `status=completed`, `expires_at` в будущем.                         |
|                | 2. Иначе 409 `export_not_ready_error` / 410 `export_expired_error` / 404.                 |
|                | 3. Отдаёт объект из MinIO/local. Query-параметров **нет** (сжатие задано при create).     |
| **Параметры**  | `scope_id`, `job_id` — path                                                               |
| **Заголовки**  | `Content-Type` по имени файла; `Content-Encoding: gzip` для не-zip `.gz`;                 |
|                | `Content-Disposition: attachment`; `Cache-Control: private, no-store`;                    |
|                | опционально `X-Content-SHA256`                                                            |

---

| Kind                         | Код | Описание                              |
|------------------------------|-----|---------------------------------------|
|                              | 200 | Файл                                  |
| auth_error                   | 401 | Не авторизован                        |
| permission_denied_error      | 403 | Недостаточно прав API key             |
| api_key_scope_mismatch_error | 403 | API key / scope mismatch              |
| job_not_found_error          | 404 | Job не найден                         |
| export_not_ready_error       | 409 | Job ещё не completed                  |
| export_expired_error         | 410 | TTL download истёк                    |
| server_error                 | 500 | Внутренняя ошибка                     |

---

<details open>
<summary><b>Пример запроса</b></summary>

```http
GET /api/v1/transfer/10000001/exports/550e8400-e29b-41d4-a716-446655440000/download
Authorization: Bearer <access_token>
```

</details>

<details open>
<summary><b>Пример ответа (заголовки)</b></summary>

```http
HTTP/1.1 200 OK
Content-Type: text/csv; charset=utf-8
Content-Encoding: gzip
Content-Disposition: attachment; filename*=UTF-8''shorts-scope-10000001-20260721.csv.gz
X-Content-SHA256: e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
Cache-Control: private, no-store
```

</details>
