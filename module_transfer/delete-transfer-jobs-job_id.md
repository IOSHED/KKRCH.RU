<a id="delete-transfer-jobs-job_id"></a>

### <span style="background:#E53935;padding:5px">DELETE</span> `/transfer/{scope_id:int}/jobs/{job_id:uuid}`

|                | Описание                                                                                  |
|----------------|-------------------------------------------------------------------------------------------|
| **Назначение** | Отмена активного export/import job                                                        |
| **Логика**     | 1. Job в `pending` или `running` → перевод в `cancelled`, worker получает сигнал stop.      |
|                | 2. Частично записанный export-файл удаляется из storage.                                  |
|                | 3. Import: откат **не выполняется** — уже созданные shorts остаются (см. note).             |
|                | 4. Job в terminal state → 409 `job_not_cancellable_error`.                                  |
| **Параметры**  | `scope_id:int`, `job_id:uuid` — path                                                      |

> **Import cancel:** транзакции commit-ятся батчами; отмена останавливает
> дальнейшие строки, но не удаляет уже импортированные ссылки. Для атомарного
> import используйте `dry_run=true`, затем повтор без dry_run.

---

| Kind                         | Код | Описание                    |
|------------------------------|-----|-----------------------------|
|                              | 204 | Job отменён                 |
| auth_error                   | 401 | Не авторизован              |
| job_not_found_error          | 404 | Job не найден               |
| job_not_cancellable_error    | 409 | Job уже завершён            |
| server_error                 | 500 | Внутренняя ошибка           |

---

<details open>
<summary><b>Пример запроса</b></summary>

```http
DELETE /api/v1/transfer/10000001/jobs/550e8400-e29b-41d4-a716-446655440000
Authorization: Bearer <access_token>
```

</details>
