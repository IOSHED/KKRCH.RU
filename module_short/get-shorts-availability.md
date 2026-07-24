<a id="get-shorts-availability"></a>

### <span style="background:#7CB342;padding:5px">GET</span> `/shorts/availability`

|                | Описание                                                                                                  |
|----------------|-----------------------------------------------------------------------------------------------------------|
| **Назначение** | Проверяет доступность пары `(subdomain, short_name)`                                                      |
| **Логика**     | 1. Валидирует `short_name` и опциональный `subdomain`                                                     |
|                | 2. Redis bloom: если ключ есть и ответ «точно свободно» → `available: true` без Postgres                  |
|                | 3. Иначе (ключ bloom отсутствует / «возможно занято» / Redis недоступен) — точный SELECT в Postgres       |
|                | &nbsp;&nbsp;&nbsp;по unique `idx_shorts_resolve_*`; PG bloom (`idx_shorts_name_bloom`) — prefilter планировщика |
|                | 4. `available: false`, если пара уже занята                                                               |
| **Параметры**  | `short_name:str` - имя ссылки                                                                             |
|                | `subdomain:str?` - поддомен                                                                               |
| **Кеш**        | Redis bloom; при miss/потере ключа — Postgres (unique + PG bloom index)                                   |

---

| Ответ                       | Код | Описание                                         |
|-----------------------------|-----|--------------------------------------------------|
|                             | 200 | Результат проверки доступности                   |
| short_name_validation_error | 400 | Ошибка валидации шаблона для short_name          |
| subdomain_validation_error  | 400 | Ошибка валидации шаблона для subdomain           |
| validation_error            | 400 | Ошибка валидации                                 |
| auth_error                  | 401 | Не авторизован                                   |
| subdomain_not_found_error   | 404 | Не найден subdomain                              |
| conflict_error              | 409 | Уже существует такая пара subdomain + short_name |
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

