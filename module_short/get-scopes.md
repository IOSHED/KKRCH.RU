<a id="get-scopes"></a>

### <span style="background:#7CB342;padding:5px">GET</span> `/scopes`

|                | Описание                                                                          |
|----------------|-----------------------------------------------------------------------------------|
| **Назначение** | Возвращает список доступных scope                                                 |
| **Логика**     | 1. Personal: `owner_user_id = me`.                                                |
|                | 2. Company owner: все scope своих компаний.                                       |
|                | 3. Участник: только scopes из `scope_members` (принятый invite).                  |
|                | 4. Фильтры `owner_type` / `company_id` сужают выдачу.                             |
| **Параметры**  | `owner_type:str?` - `personal` или `company`, `company_id:int?` - фильтр компании |
| **Кеш**        | `scopes:list:{user}:{v}:{filters_hash}`, TTL 60 сек (cache-aside)                 |

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
      "billing_plan": {
        "id": "FREE",
        "max_shorts": 10,
        "max_subdomains": 0,
        "max_seats": null,
        "transfer_daily_bytes": 0,
        "stats_click_retention_days": 30
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
      "billing_plan": {
        "id": "BUSINESS",
        "max_shorts": 10000,
        "max_subdomains": 10,
        "max_seats": 10,
        "transfer_daily_bytes": 104857600,
        "stats_click_retention_days": 90
      },
      "created_at": "2024-06-01T12:00:00Z",
      "update_at": "2024-06-01T12:00:00Z"
    }
  ]
}
```

`billing_plan` — тариф **billing owner** (личный owner или owner компании). Участник corp-scope видит лимиты владельца, а не своего личного плана.
</details>
