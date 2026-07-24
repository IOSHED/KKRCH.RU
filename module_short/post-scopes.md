<a id="post-scopes"></a>

### <span style="background:#42A5F5;padding:5px">POST</span> `/scopes`

|                      | Описание                                                                   |
|----------------------|----------------------------------------------------------------------------|
| **Назначение**       | Создает scope                                                              |
| **Логика**           | 1. Валидирует входные данные                                               |
|                      | 2. Создает scope                                                           |
| **Параметры**        | `company_id:int?` - привязка к компании, иначе привязка к личному аккаунту |
| **Инвалидация кеша** | `INCR scopes:list:{owner}:v`                                               |

---

| Ответ                 | Код | Описание                                                    |
|-----------------------|-----|-------------------------------------------------------------|
| success               | 201 | Scope создан                                                |
| validation_error      | 400 | Ошибка валидации                                            |
| auth_error            | 401 | Не авторизован                                              |
| scope_not_payed_error | 402 | Запрошены n-ный scope без его имения (ограничены подпиской) |
| server_error          | 500 | Внутренняя ошибка сервера                                   |

---

<details open>
<summary><b>Пример запроса</b></summary>

```json
{
  "name": "Marketing",
  "description": "Main marketing scope",
  "company_id": 1001
}
```

</details>

<details open>
<summary><b>Пример ответа</b></summary>

```json
{
  "scope": {
    "id": 10000001,
    "name": "Marketing",
    "description": "Main marketing scope",
    "created_at": "2024-06-01T12:00:00Z",
    "owner": {
      "type": "company",
      "company_id": 1001
    }
  }
}
```

</details>
