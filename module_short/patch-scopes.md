<a id="patch-scopes"></a>

### <span style="background:#FFA726;padding:5px">PATCH</span> `/scopes/{scope_id:int}`

|                      | Описание                             |
|----------------------|--------------------------------------|
| **Назначение**       | Обновляет название scope             |
| **Auth**             | Bearer или X-Api-Key (`scopes:write`) |
| **Логика**           | 1. Валидирует доступ                 |
|                      | 2. Обновляет имя                     |
| **Параметры**        | `scope_id:int` - идентификатор scope |
| **Инвалидация кеша** | `INCR scopes:list:{owner}:v`         |

---

| Ответ                 | Код | Описание                  |
|-----------------------|-----|---------------------------|
| success               | 200 | Scope обновлен            |
| validation_error      | 400 | Ошибка валидации          |
| auth_error            | 401 | Не авторизован            |
| permission_denied_error       | 403 | Недостаточно прав у API key |
| api_key_scope_mismatch_error  | 403 | API key привязан к другому scope |
| scope_not_found_error | 404 | Scope не найден           |
| server_error          | 500 | Внутренняя ошибка сервера |

---

<details open>
<summary><b>Пример запроса</b></summary>

```json
{
  "name": "Marketing v2",
  "description": "Updated description for marketing scope"
}
```

</details>

<details open>
<summary><b>Пример ответа</b></summary>

```json
{
  "scope": {
    "id": 10000001,
    "name": "Marketing v2",
    "description": "Updated description for marketing scope"
  }
}
```

</details>

