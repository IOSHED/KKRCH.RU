<a id="delete-shorts-bulk"></a>

### <span style="background:#EF5350;padding:5px">DELETE</span> `/shorts/bulk`

|                      | Описание                                                                                |
|----------------------|-----------------------------------------------------------------------------------------|
| **Назначение**       | Удаляет короткие ссылки                                                                 |
| **Auth**             | Bearer или X-Api-Key                                                                    |
| **Логика**           | 0. Если число элементов превышает лимит (`bulk.max_items`) → 400 `too_many_items_error` |
|                      | 1. Валидирует доступ                                                                    |
|                      | 2. Удаляет ссылку, её метаданные и raw-клики в Redis/Postgres                           |
|                      | 3. Per-short агрегированная статистика не удаляется                                     |
| **Инвалидация кеша** | `INCR shorts:scope:{scope}:v`, очистка `short:resolve:{subdomain}:{short_name}`         |

---

| Ответ                 | Код | Описание                                                                                         |
|-----------------------|-----|--------------------------------------------------------------------------------------------------|
|                       | 204 | Все ссылки удалены                                                                               |
|                       | 207 | Частично удалены ссылки, в ответе будет информация о том, какие ссылки были удалены, а какие нет |
| too_many_items_error  | 400 | Превышен лимит элементов в bulk-запросе (`bulk.max_items`)                                       |
| auth_error            | 401 | Не авторизован                                                                                   |
| permission_denied_error       | 403 | Недостаточно прав у API key |
| api_key_scope_mismatch_error  | 403 | API key привязан к другому scope |
| short_not_found_error | 404 | Ссылка не найдена                                                                                |
| server_error          | 500 | Внутренняя ошибка сервера                                                                        |

---

<details open>
<summary><b>Пример запроса</b></summary>

```json
{
  "short_ids": [
    10000001,
    10000002
  ]
}
```

</details>

<details open>
<summary><b>Пример ответа</b></summary>

```json
{
  "errors": [
    {
      "index": 1,
      "kind": "short_not_found_error",
      "reason": "Short with id '10000002' not found"
    }
  ]
}
```

> `errors` может содержать все переданные ссылки, но при этом ответ все еще 207

</details>

