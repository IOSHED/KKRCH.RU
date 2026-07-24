<a id="get-auth-profile"></a>

### <span style="background:#7CB342;padding:5px">GET</span> `/auth/profile`

|                | Описание                                                                              |
|----------------|---------------------------------------------------------------------------------------|
| **Назначение** | Получение профиля текущего пользователя                                               |
| **Логика**     | 1. Валидирует access_token в заголовке `Authorization`.                               |
|                | 2. Извлекает `subscription` и `roles` прямо из access token — без запроса в Postgres. |
|                | 3. Запрашивает базовые данные пользователя из таблицы `users`.                        |
|                | 4. Получает список связанных OAuth-провайдеров (`oauth_accounts`).                    |
|                | 5. Загружает текущий тарифный план из `subscription_plans` по `users.subscription`.   |
|                | 6. Определяет следующий тариф по стоимости (+1 уровень апгрейда по `price_rub`).      |
|                | 7. Возвращает профиль, текущий план и `upgrade_plan` (`null` на максимальном тарифе). |

---

| Kind                 | Код | Описание                          |
|----------------------|-----|-----------------------------------|
|                      | 200 | Профиль пользователя              |
| auth_error           | 401 | Нет или некорректный access token |
| user_not_found_error | 404 | Пользователь не найден            |
| server_error         | 500 | Внутренняя ошибка сервера         |

---

<details open>
<summary><b>Пример ответа</b></summary>

```json
{
  "user": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "auth_provider": "YANDEX",
    "email": "user@example.com",
    "display_name": "John",
    "subscription": "FREE",
    "roles": [],
    "created_at": "2025-01-15T10:00:00Z",
    "updated_at": "2025-01-17T10:00:00Z"
  },
  "subscription_plan": {
    "id": "FREE",
    "price_rub": 0,
    "max_scopes": 1,
    "max_subdomains": 0,
    "max_shorts": 10,
    "max_seats": null,
    "stats_click_retention_days": 30,
    "is_corporate": false,
    "description": "Бесплатная подписка для ознакомления"
  },
  "upgrade_plan": {
    "id": "PERSONAL",
    "price_rub": 100,
    "max_scopes": 2,
    "max_subdomains": 0,
    "max_shorts": 300,
    "max_seats": null,
    "stats_click_retention_days": 90,
    "is_corporate": false,
    "description": "Подписка для индивидуального использования"
  }
}
```

</details>

