<a id="patch-folders"></a>

### <span style="background:#FFA726;padding:5px">PATCH</span> `/folders/{folder_id:uuid}`

|                      | Описание                               |
|----------------------|----------------------------------------|
| **Назначение**       | Обновляет параметры папки              |
| **Auth**             | Bearer или X-Api-Key                   |
| **Логика**           | 1. Валидирует доступ                   |
|                      | 2. Обновляет все переданные поля       |
| **Параметры**        | `folder_id:uuid` - идентификатор папки |
| **Инвалидация кеша** | `INCR folders:scope:{scope}:v`         |

---

| Ответ                  | Код | Описание                         |
|------------------------|-----|----------------------------------|
|                        | 200 | Папка обновлена                  |
| name_validation_error  | 400 | Ошибка валидации name            |
| color_validation_error | 400 | Ошибка валидации color           |
| validation_error       | 400 | Ошибка валидации прочих полей    |
| auth_error             | 401 | Не авторизован                   |
| permission_denied_error       | 403 | Недостаточно прав у API key |
| api_key_scope_mismatch_error  | 403 | API key привязан к другому scope |
| folder_not_found_error | 404 | Папка не найдена или нет доступа |
| server_error           | 500 | Внутренняя ошибка сервера        |

---

<details open>
<summary><b>Пример запроса</b></summary>

```json
{
  "name": "folder 1 updated",
  "color": "#3357FF",
  "parent_id": "def12345-e89b-12d3-a456-426614174000"
}
```

> Поля, которых нет в JSON — не меняются. `"parent_id": null` перемещает папку в корень.

</details>

<details open>
<summary><b>Пример ответа</b></summary>

```json
{
  "folder": {
    "id": "abc12345-e89b-12d3-a456-426614174000",
    "name": "folder 1 updated",
    "color": "#3357FF",
    "parent_id": "def12345-e89b-12d3-a456-426614174000",
    "created_at": "2024-06-01T12:00:00Z"
  }
}
```

</details>

