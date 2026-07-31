<a id="post-subdomains"></a>

### <span style="background:#42A5F5;padding:5px">POST</span> `/subdomains`

|                      | Описание                                                                            |
|----------------------|-------------------------------------------------------------------------------------|
| **Назначение**       | Создаёт subdomain (или восстанавливает ранее удалённый)                             |
| **Auth**             | Bearer или X-Api-Key                                                                  |
| **Логика**           | 1. Валидирует `name` по правилам домена                                             |
|                      | 2. Первый label `routing_cname_target` (default `edge`) → 400                       |
|                      | 3. Если поддомен уже существует и активен — 409                                     |
|                      | 4. Если поддомен был удалён (`deleted_at IS NOT NULL`) и закреплён за тем же scope: |
|                      | &nbsp;&nbsp;&nbsp;- Восстанавливает запись (`deleted_at = NULL`)                    |
|                      | &nbsp;&nbsp;&nbsp;- Реактивирует ссылки с `inactive_reason = 'subdomain_deleted'`   |
|                      | &nbsp;&nbsp;&nbsp;- Ссылки, отключённые пользователем вручную, **не затрагиваются** |
|                      | 5. Иначе — создаёт новую запись с привязкой к указанному `scope_id`                 |
|                      | 6. Инвалидирует Redis-версию списка subdomains                                      |
| **Параметры**        | `scope_id:int` - scope, к которому привязывается поддомен (определяет владельца)    |
| **Инвалидация кеша** | `INCR subdomains:list:{owner}:v`                                                    |

> Owner поддомена (user/company) выводится из `scope_id` через JOIN со `scopes`
> (`scopes.owner_user_id` / `scopes.company_id`). Отдельного параметра
> `company_id` у поддомена больше нет.

---

| Ответ                     | Код | Описание                                                                |
|---------------------------|-----|-------------------------------------------------------------------------|
| success                   | 201 | Subdomain создан                                                        |
| subdomain_validation_error | 400 | Невалидное имя / зарезервировано (`edge` и т.п.)                   |
| validation_error          | 400 | Ошибка валидации                                                        |
| auth_error                | 401 | Не авторизован                                                          |
| permission_denied_error       | 403 | Недостаточно прав у API key |
| api_key_scope_mismatch_error  | 403 | API key привязан к другому scope |
| subdomain_not_payed_error | 402 | Запрошены n-ный премиум subdomain без его имения (ограничены подпиской) |
| conflict_error            | 409 | Subdomain уже занят                                                     |
| server_error              | 500 | Внутренняя ошибка сервера                                               |

---

<details open>
<summary><b>Пример запроса</b></summary>

```json
{
  "name": "my-short-link",
  "scope_id": 42
}
```

</details>

<details open>
<summary><b>Пример ответа</b></summary>

```json
{
  "subdomain": {
    "name": "my-short-link",
    "deleted_at": null,
    "owner": {
      "type": "company",
      "company_id": 1001
    }
  }
}
```

</details>
