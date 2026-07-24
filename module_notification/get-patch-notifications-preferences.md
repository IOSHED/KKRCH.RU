<a id="get-patch-notifications-preferences"></a>

### <span style="background:#7CB342;padding:5px">GET</span> / <span style="background:#FB8C00;padding:5px">PATCH</span> `/notifications/preferences`

|                | Описание                                                              |
|----------------|-----------------------------------------------------------------------|
| **Назначение** | Настройки inbox; opt-out promo (152-ФЗ)                               |
| **Логика**     | GET — текущие prefs; PATCH — частичное обновление.                    |
|                | `in_app_promo` default **false** (opt-in).                            |

---

| Kind         | Код | Описание       |
|--------------|-----|----------------|
|              | 200 | Prefs          |
| auth_error   | 401 | Не авторизован |
| server_error | 500 | Ошибка         |

---

<details open>
<summary><b>Пример</b></summary>

```json
{
  "in_app_support": true,
  "in_app_release": true,
  "in_app_promo": false
}
```

</details>
