<a id="get-auth-subscription_plans"></a>

### <span style="background:#7CB342;padding:5px">GET</span> `/auth/subscription_plans`

|                | Описание                                                                                         |
|----------------|--------------------------------------------------------------------------------------------------|
| **Назначение** | Публичный каталог тарифных планов для лендинга / Upgrade UI                                      |
| **Логика**     | 1. Читает записи из `subscription_plans`, у которых **`is_view = true`**.                        |
|                | 2. Сортирует планы по возрастанию `price_rub` (при равной цене — по `id`).                       |
|                | 3. Возвращает поля плана **без** внутреннего `is_view` (поле только фильтрует выборку).          |
|                | 4. Ответ кешируется браузером: `Cache-Control: public, max-age=86400`.                           |
| **Важно**      | `is_view` **не** влияет на `GET /auth/profile` (`subscription_plan` / `upgrade_plan`) и лимиты — |
|                | там план читается по `users.subscription` целиком.                                               |

На запуске без платёжного модуля в каталоге виден только `FREE_PLUS` (`is_view = true`). Остальные тарифы в БД остаются и отдаются в профиле / используются в проверках лимитов.

---

| Kind         | Код | Описание                   |
|--------------|-----|----------------------------|
|              | 200 | Справочник тарифных планов |
| server_error | 500 | Внутренняя ошибка сервера  |

---

<details open>
<summary><b>Пример ответа</b></summary>

```json
{
  "plans": [
    {
      "id": "FREE_PLUS",
      "price_rub": 0,
      "max_scopes": 2,
      "max_subdomains": 0,
      "max_shorts": 300,
      "max_seats": null,
      "stats_click_retention_days": 90,
      "is_corporate": false,
      "description": "Бесплатный расширенный тариф на период запуска (без оплаты)"
    }
  ]
}
```

</details>
