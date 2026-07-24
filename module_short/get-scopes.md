<a id="get-scopes"></a>

### <span style="background:#7CB342;padding:5px">GET</span> `/scopes`

|                | Описание                                                                          |
|----------------|-----------------------------------------------------------------------------------|
| **Назначение** | Возвращает список доступных scope                                                 |
| **Логика**     | 1. Возвращает scope пользователя/компании                                         |
| **Параметры**  | `owner_type:str?` - `personal` или `company`, `company_id:int?` - фильтр компании |
| **Кеш**        | `scopes:list:{owner}:{v}:{filters_hash}`, TTL 60 сек (cache-aside)                |

---

| Ответ        | Код | Описание                  |
|--------------|-----|---------------------------|
| success      | 200 | Список scope              |
| auth_error   | 401 | Не авторизован            |
| server_error | 500 | Внутренняя ошибка сервера |

---

<details open>
<summary><b>Пример ответа</b></summary>

```json
{
  "scopes": [
    {
      "id": 10000001,
      "name": "Marketing",
      "description": "Main marketing scope",
      "owner": {
        "type": "personal"
      },
      "created_at": "2024-06-01T12:00:00Z",
      "update_at": "2024-06-01T12:00:00Z"
    },
    {
      "id": 10000002,
      "name": "Brand",
      "description": "Company scope",
      "owner": {
        "type": "company",
        "company_id": 1001
      },
      "created_at": "2024-06-01T12:00:00Z",
      "update_at": "2024-06-01T12:00:00Z"
    }
  ]
}
```

</details>
