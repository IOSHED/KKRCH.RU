<a id="get-custom-domains"></a>

### <span style="background:#7CB342;padding:5px">GET</span> `/custom-domains`

|                | Описание                                                                                                                          |
|----------------|-----------------------------------------------------------------------------------------------------------------------------------|
| **Назначение** | Список собственных доменов аккаунта                                                                                               |
| **Auth**       | Bearer или X-Api-Key                                                                                                              |
| **Логика**     | 1. Возвращает домены, чей `scope_id` принадлежит пользователю (JOIN со `scopes`)                                                |
|                | 2. По умолчанию — только `deleted_at IS NULL`                                                                                     |
|                | 3. `include_deleted=true` — также soft-deleted (для UI истории)                                                                 |
|                | 4. `verification_token` **не** возвращается (только при create / reissue)                                                         |
| **Параметры**  | `owner_type:str?` — `personal` \| `company`; `company_id:int?`; `scope_id:int?`; `status:str?` — фильтр по статусу; `include_deleted:bool?` |
| **Кеш**        | `custom_domains:list:{owner}:{v}:{filters_hash}`, TTL 60 сек (cache-aside)                                                        |

---

| Ответ                        | Код | Описание                  |
|------------------------------|-----|---------------------------|
| success                      | 200 | Список доменов            |
| auth_error                   | 401 | Не авторизован            |
| permission_denied_error      | 403 | Недостаточно прав у API key |
| api_key_scope_mismatch_error | 403 | API key привязан к другому scope |
| server_error                 | 500 | Внутренняя ошибка сервера |

---

<details open>
<summary><b>Пример ответа</b></summary>

```json
{
  "custom_domains": [
    {
      "domain": "go.company.ru",
      "scope_id": 42,
      "status": "active",
      "verified_at": "2026-07-30T10:15:00Z",
      "routing_checked_at": "2026-07-30T10:15:00Z",
      "verification_expires_at": null,
      "last_check_at": "2026-07-30T10:15:00Z",
      "last_check_error": null,
      "created_at": "2026-07-30T09:00:00Z",
      "deleted_at": null,
      "owner": {
        "type": "company",
        "company_id": 1001
      }
    },
    {
      "domain": "links.brand.ru",
      "scope_id": 42,
      "status": "pending_verification",
      "verified_at": null,
      "routing_checked_at": null,
      "verification_expires_at": "2026-08-02T09:00:00Z",
      "last_check_at": "2026-07-30T09:30:00Z",
      "last_check_error": "TXT record not found",
      "created_at": "2026-07-30T09:00:00Z",
      "deleted_at": null,
      "owner": {
        "type": "company",
        "company_id": 1001
      }
    }
  ],
  "limit": 3
}
```

</details>
