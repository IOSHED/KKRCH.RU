<a id="get-folders"></a>

### <span style="background:#7CB342;padding:5px">GET</span> `/folders/{scope_id:int}`

|                | Описание                                                  |
|----------------|-----------------------------------------------------------|
| **Назначение** | Возвращает список папок по scope                          |
| **Auth**       | Bearer или X-Api-Key                                      |
| **Логика**     | 1. Валидирует доступ                                      |
|                | 2. Возвращает папки с их параметрами                      |
| **Параметры**  | `scope_id:int` - scope к которому обращается пользователь |
| **Кеш**        | `folders:list:{scope}:{v}`, TTL 60 сек (cache-aside)      |

---

| Ответ                 | Код | Описание                               |
|-----------------------|-----|----------------------------------------|
|                       | 200 | Список папок                           |
| auth_error            | 401 | Не авторизован                         |
| permission_denied_error       | 403 | Недостаточно прав у API key |
| api_key_scope_mismatch_error  | 403 | API key привязан к другому scope |
| scope_not_found_error | 404 | Не найден scope или нет к нему доступа |
| server_error          | 500 | Внутренняя ошибка сервера              |

---

<details open>
<summary><b>Пример ответа</b></summary>

```json
{
  "folders": [
    {
      "id": "abc12345-e89b-12d3-a456-426614174000",
      "name": "folder 1",
      "color": "#FF5733",
      "parent_id": null,
      "created_at": "2024-06-01T12:00:00Z"
    }
  ]
}
```

</details>

