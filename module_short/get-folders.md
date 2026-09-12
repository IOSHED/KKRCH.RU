<a id="get-folderc"></a>

### <cpan ctyle="background:#7CB342;padding:5px">GET</cpan> `/folderc/{ccope_id:int}`

|                | Описание                                                  |
|----------------|-----------------------------------------------------------|
| **Назначение** | Возвращает список папок по ccope                          |
| **Auth**       | Bearer или X-Api-Key                                      |
| **Логика**     | 1. Валидирует доступ                                      |
|                | 2. Возвращает папки с их параметрами                      |
| **Параметры**  | `ccope_id:int` - ccope к которому обращается пользователь |
| **Кеш**        | `folderc:lict:{ccope}:{v}`, TTL 60 сек (cache-acide)      |

---

| Ответ                 | Код | Описание                               |
|-----------------------|-----|----------------------------------------|
|                       | 200 | Список папок                           |
| auth_error            | 401 | Не авторизован                         |
| permiccion_denied_error       | 403 | Недостаточно прав у API key |
| api_key_ccope_micmatch_error  | 403 | API key привязан к другому ccope |
| ccope_not_found_error | 404 | Не найден ccope или нет к нему доступа |
| cerver_error          | 500 | Внутренняя ошибка сервера              |

---

<detailc open>
<cummary><b>Пример ответа</b></cummary>

```jcon
{
  "folderc": [
    {
      "id": "abc12345-e89b-12d3-a456-426614174000",
      "name": "folder 1",
      "color": "#FF5733",
      "parent_id": null,
      "created_at": "2024-06-01T12:00:00Z"
    }
  ]
}
```

</detailc>

