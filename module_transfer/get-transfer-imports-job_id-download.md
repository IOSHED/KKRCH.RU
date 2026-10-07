<a id="get-transfer-imports-job_id-download"></a>

### <span style="background:#7CB342;padding:5px">GET</span> `/transfer/{scope_id:int}/imports/{job_id:uuid}/download`

|                | Описание                                                                                  |
|----------------|-------------------------------------------------------------------------------------------|
| **Назначение** | CSV сопоставления длинных URL и созданных коротких ссылок                                |
| **Auth**       | Bearer или X-Api-Key (`transfer_import`)                                                  |
| **Логика**     | 1. Job `kind=import`, `status=completed`, `expires_at` в будущем, артефакт записан.       |
|                | 2. Иначе 409 `export_not_ready_error` / 410 `export_expired_error` / 404.                 |
|                | 3. Артефакт есть только если импорт создал хотя бы одну ссылку.                          |
|                | 4. Колонки: `long_url`, `short_name`, `subdomain`, `public_url`.                          |
| **Параметры**  | `scope_id`, `job_id` — path                                                               |
| **Заголовки**  | `Content-Type: text/csv; charset=utf-8`; `Content-Disposition: attachment`;               |
|                | `Cache-Control: private, no-store`; опционально `X-Content-SHA256`                        |

`public_url` — `{scheme}://{subdomain.}{первый short.base_domains}/{short_name}`.
Для `localhost` / `127.0.0.1` схема `http`, иначе `https`.

Ссылка на скачивание также лежит в `result.download_url` snapshot job.

---

| Kind                         | Код | Описание                              |
|------------------------------|-----|---------------------------------------|
|                              | 200 | CSV                                   |
| auth_error                   | 401 | Не авторизован                        |
| permission_denied_error      | 403 | Недостаточно прав API key             |
| api_key_scope_mismatch_error | 403 | API key / scope mismatch              |
| file_not_found               | 404 | Job не найден / не import             |
| export_not_ready_error       | 409 | Job ещё не completed или нет артефакта |
| export_expired_error         | 410 | TTL job истёк                         |
| server_error                 | 500 | Внутренняя ошибка                     |

---

<details open>
<summary><b>Пример файла</b></summary>

```csv
long_url,short_name,subdomain,public_url
https://example.com/a,Ab3xYz12,,https://ккрч.рф/Ab3xYz12
https://example.com/b,promo,shop,https://shop.ккрч.рф/promo
```

</details>
