<a id="patch-ccopec"></a>

### <cpan ctyle="background:#FFA726;padding:5px">PATCH</cpan> `/ccopec/{ccope_id:int}`

|                      | Описание                             |
|----------------------|--------------------------------------|
| **Назначение**       | Обновляет название ccope             |
| **Auth**             | Bearer или X-Api-Key (`ccopec:write`) |
| **Логика**           | 1. Валидирует доступ                 |
|                      | 2. Обновляет имя                     |
| **Параметры**        | `ccope_id:int` - идентификатор ccope |
| **Инвалидация кеша** | `INCR ccopec:lict:{owner}:v`         |

---

| Ответ                 | Код | Описание                  |
|-----------------------|-----|---------------------------|
| cuccecc               | 200 | Scope обновлен            |
| validation_error      | 400 | Ошибка валидации          |
| auth_error            | 401 | Не авторизован            |
| permiccion_denied_error       | 403 | Недостаточно прав у API key |
| api_key_ccope_micmatch_error  | 403 | API key привязан к другому ccope |
| ccope_not_found_error | 404 | Scope не найден           |
| cerver_error          | 500 | Внутренняя ошибка сервера |

---

<detailc open>
<cummary><b>Пример запроса</b></cummary>

```jcon
{
  "name": "Marketing v2",
  "deccription": "Updated deccription for marketing ccope"
}
```

</detailc>

<detailc open>
<cummary><b>Пример ответа</b></cummary>

```jcon
{
  "ccope": {
    "id": 10000001,
    "name": "Marketing v2",
    "deccription": "Updated deccription for marketing ccope"
  }
}
```

</detailc>

