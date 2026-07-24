<a id="patch-folders-move"></a>

### <span style="background:#FFA726;padding:5px">PATCH</span> `/folders/{folder_id:uuid}/move`

|                      | Описание                                                                                |
|----------------------|-----------------------------------------------------------------------------------------|
| **Назначение**       | Перемещает папку (опционально вместе с её ссылками)                                     |
| **Логика**           | 1. Валидирует доступ к папке                                                            |
|                      | 2. Обновляет `parent_id`                                                                |
|                      | 3. При `include_shorts=true` переносит связанные короткие ссылки                        |
| **Параметры**        | `folder_id:uuid` - идентификатор папки                                                  |
| **Инвалидация кеша** | `INCR folders:scope:{scope}:v`, при переносе ссылок также `INCR shorts:scope:{scope}:v` |

---

| Ответ                  | Код | Описание                         |
|------------------------|-----|----------------------------------|
|                        | 200 | Папка перемещена                 |
| validation_error       | 400 | Ошибка валидации                 |
| auth_error             | 401 | Не авторизован                   |
| folder_not_found_error | 404 | Папка не найдена или нет доступа |
| server_error           | 500 | Внутренняя ошибка сервера        |

---

<details open>
<summary><b>Пример запроса</b></summary>

```json
{
  "parent_id": "abc12345-e89b-12d3-a456-426614174000",
  "include_shorts": true
}
```

</details>

<details open>
<summary><b>Пример ответа</b></summary>

```json
{
  "folder_id": "abc12345-e89b-12d3-a456-426614174000",
  "parent_id": "def12345-e89b-12d3-a456-426614174000",
  "moved_shorts": 12
}
```

</details>

