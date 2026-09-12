<a id="get-shorts-availability"></a>

### <span style="background:#7CB342;padding:5px">GET</span> `/shorts/availability`

|                | Описание                                                                                                  |
|----------------|-----------------------------------------------------------------------------------------------------------|
| **Назначение** | Проверяет доступность пары `(subdomain|custom_domain, short_name)`                                |
| **Auth**       | Bearer или X-Api-Key                                                                                      |
| **Логика**     | 1. Валидирует `short_name` и host (`subdomain` XOR `custom_domain`)                                       |
|                | 2. Redis bloom: если ключ есть и ответ «точно свободно» → `available: true` без Postgres                  |
|                | 3. Иначе (ключ bloom отсутствует / «возможно занято» / Redis недоступен) — точный SELECT в Postgres       |
|                | &nbsp;&nbsp;&nbsp;по unique `idx_shorts_resolve_*` / `idx_shorts_custom_domain_short_name`                 |
|                | 4. `available: false`, если пара уже занята                                                               |
| **Параметры**  | `short_name:str` - имя ссылки                                                                             |
|                | `subdomain:str?` - поддомен платформы                                                                     |
|                | `custom_domain:str?` - собственный домен (FQDN); взаимоисключающ с `subdomain`                            |
| **Кеш**        | Redis bloom; при miss/потере ключа — Postgres (unique + PG bloom index)                                   |

---

| Ответ                       | Код | Описание                                         |
|-----------------------------|-----|--------------------------------------------------|
|                             | 200 | Результат проверки доступности                   |
| short_name_validation_error | 400 | Ошибка валидации шаблона для short_name          |
| subdomain_template_validation  | 400 | Ошибка валидации шаблона для subdomain           |
| validation_error            | 400 | Ошибка валидации                                 |
| auth_error                  | 401 | Не авторизован                                   |
| permission_denied_error       | 403 | Недостаточно прав у API key |
| api_key_scope_mismatch_error  | 403 | API key привязан к другому scope |
| subdomain_not_found_error   | 404 | Не найден subdomain                              |
| host_short_name_conflict              | 409 | Уже существует такая пара subdomain + short_name |
| server_error                | 500 | Внутренняя ошибка сервера                        |

---

<details open>
<summary><b>Пример ответа</b></summary>

```json
{
  "short_name": "custom-slug",
  "subdomain": "my-short-link",
  "available": true
}
```

</details>

