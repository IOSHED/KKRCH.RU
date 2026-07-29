<a id="post-folders-bulk-create"></a>

### <span style="background:#42A5F5;padding:5px">POST</span> `/folders/{scope_id:int}/bulk`

|                      | Описание                                                                                |
|----------------------|-----------------------------------------------------------------------------------------|
| **Назначение**       | Позволяет массово создавать папки                                                       |
| **Auth**             | Bearer или X-Api-Key                                                                    |
| **Логика**           | 0. Если число элементов превышает лимит (`bulk.max_items`) → 400 `too_many_items_error` |
|                      | 1. Каждый элемент проходит независимую валидацию; невалидные → в `errors` с индексом    |
|                      | 2. Bulk INSERT всех валидных папок в рамках одной транзакции                            |
|                      | 3. После commit инвалидируется Redis-версия списка папок scope                          |
| **Параметры**        | `scope_id:int` - индификатор scope                                                      |
| **Инвалидация кеша** | `INCR folders:scope:{scope}:v`                                                          |

---

| Ответ                  | Код | Описание                                                                                       |
|------------------------|-----|------------------------------------------------------------------------------------------------|
|                        | 201 | Созданы все папки                                                                              |
|                        | 207 | Частично созданы папки, в ответе будет информация о том, какие папки были созданы, а какие нет |
| too_many_items_error   | 400 | Превышен лимит элементов в bulk-запросе (`bulk.max_items`)                                     |
| name_validation_error  | 400 | Ошибка валидации name                                                                          |
| color_validation_error | 400 | Ошибка валидации color                                                                         |
| validation_error       | 400 | Ошибка валидации прочих полей                                                                  |
| auth_error             | 401 | Не авторизован                                                                                 |
| permission_denied_error       | 403 | Недостаточно прав у API key |
| api_key_scope_mismatch_error  | 403 | API key привязан к другому scope |
| scope_not_found_error  | 404 | Не найден scope или нет к нему доступа                                                         |
| server_error           | 500 | Внутренняя ошибка сервера                                                                      |

---

<details open>
<summary><b>Пример запроса</b></summary>

```json
{
  "folders": [
    {
      // Необязательное поле, по умолчанию рандомный uuid
      "id": "abc12345-e89b-12d3-a456-426614174000",
      // Отображаемое название папки, может быть не уникальным
      "name": "folder 1",
      // Необязательное поле, по умолчанию голубая
      "color": "#FF5733",
      // Необязательное поле, по умолчанию null
      "parent_id": null
    },
    {
      "name": "folde\r 2",
      "parent_id": "abc12345-e89b-12d3-a456-426614174000"
    }
  ]
}
```

</details>

<details open>
<summary><b>Пример ответа</b></summary>

```json
{
  "folders": [
    {
      "id": "abc12345-e89b-12d3-a456-426614174000",
      "name": "folder1",
      "parent_id": null,
      "created_at": "2024-06-01T12:00:00Z"
    }
  ],
  "errors": [
    {
      "index": 1,
      "kind": "name_validation_error",
      "reason": "Folder name contains invalid characters"
    }
  ]
}
```

</details>

