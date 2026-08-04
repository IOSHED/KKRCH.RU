<a id="delete-folders-bulk"></a>

### <span style="background:#EF5350;padding:5px">DELETE</span> `/folders/bulk`

|                      | Описание                                                                                              |
|----------------------|-------------------------------------------------------------------------------------------------------|
| **Назначение**       | Удаляет папки                                                                                         |
| **Auth**             | Bearer или X-Api-Key                                                                                  |
| **Логика**           | 0. Если число элементов превышает лимит (`bulk.max_items`) → 400 `too_many_items_error`               |
|                      | 1. Валидирует доступ (`folders:delete`; при `on_shorts=delete` ApiKey также нужен `shorts:delete`)    |
|                      | 2. Обрабатывает ссылки внутри удаляемых папок согласно `on_shorts`                                    |
|                      | ↳ `move_to_parent` (default) — переносит в ближайшего предка вне списка удаления (или в корень)       |
|                      | ↳ `move_to_root` — сбрасывает `folder_id` в `NULL` (корень scope)                                     |
|                      | ↳ `delete` — удаляет ссылки                                                                           |
|                      | 3. Удаляет папки                                                                                      |
| **Параметры**        | `folder_ids:uuid[]` — идентификаторы папок                                                            |
|                      | `on_shorts:enum` — `move_to_parent` \| `move_to_root` \| `delete` (default: `move_to_parent`)         |
| **Инвалидация кеша** | `INCR folders:scope:{scope}:v`, `INCR shorts:scope:{scope}:v`; при `delete` также drop resolve-ключей |

---

| Ответ                        | Код | Описание                                                                                       |
|------------------------------|-----|------------------------------------------------------------------------------------------------|
|                              | 204 | Все папки удалены                                                                              |
|                              | 207 | Частично удалены папки, в ответе будет информация о том, какие папки были удалены, а какие нет |
| too_many_items_error         | 400 | Превышен лимит элементов в bulk-запросе (`bulk.max_items`)                                     |
| auth_error                   | 401 | Не авторизован                                                                                 |
| permission_denied_error      | 403 | Недостаточно прав у API key                                                                    |
| api_key_scope_mismatch_error | 403 | API key привязан к другому scope                                                               |
| folder_not_found_error       | 404 | Папка не найдена                                                                               |
| server_error                 | 500 | Внутренняя ошибка сервера                                                                      |

---

<details open>
<summary><b>Пример запроса</b></summary>

```json
{
  "folder_ids": [
    "abc12345-e89b-12d3-a456-426614174000",
    "def12345-e89b-12d3-a456-426614174000"
  ],
  "on_shorts": "move_to_parent"
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
      "kind": "folder_not_found_error",
      "reason": "Folder with id 'def12345-e89b-12d3-a456-426614174000' not found"
    }
  ]
}
```

</details>

