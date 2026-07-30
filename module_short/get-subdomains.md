<a id="get-subdomains"></a>

### <span style="background:#7CB342;padding:5px">GET</span> `/subdomains`

|                | Описание                                                                                                                                             |
|----------------|------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Назначение** | Возвращает список subdomains                                                                                                                         |
| **Auth**       | Bearer или X-Api-Key                                                                                                                                 |
| **Логика**     | 1. Возвращает subdomains, чей `scope_id` принадлежит пользователю (JOIN со `scopes`) — фильтрация по `owner_type` / `company_id` идёт по полям scope |
| **Параметры**  | `owner_type:str?` - `personal` или `company`, `company_id:int?` - фильтр компании                                                                    |
| **Кеш**        | `subdomains:list:{owner}:{v}:{filters_hash}`, TTL 60 сек (cache-aside)                                                                               |

---

| Ответ                        | Код | Описание                         |
|------------------------------|-----|----------------------------------|
| success                      | 200 | Список subdomains                |
| auth_error                   | 401 | Не авторизован                   |
| permission_denied_error      | 403 | Недостаточно прав у API key      |
| api_key_scope_mismatch_error | 403 | API key привязан к другому scope |
| server_error                 | 500 | Внутренняя ошибка сервера        |

---

<details open>
<summary><b>Пример ответа</b></summary>

```json
{
  "subdomains": [
    {
      "name": "my-short-link",
      "deleted_at": null,
      "owner": {
        "type": "personal"
      }
    },
    {
      "name": "brand-short-link",
      "deleted_at": "2024-06-01T12:00:00Z",
      "owner": {
        "type": "company",
        "company_id": 1001
      }
    }
  ]
}
```

</details>
