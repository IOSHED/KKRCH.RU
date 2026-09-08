<a id="get-patch-notifications-preferences"></a>

### <span style="background:#7CB342;padding:5px">GET</span> / <span style="background:#FB8C00;padding:5px">PATCH</span> `/notifications/preferences`

|                | Описание                                                              |
|----------------|-----------------------------------------------------------------------|
| **Назначение** | Настройки каналов in-app / browser / email; opt-out promo (152-ФЗ)    |
| **Логика**     | GET — текущие prefs; PATCH — частичное обновление.                    |
|                | `in_app_promo` default **false** (opt-in).                            |
|                | `email_*` / `browser_support` / `in_app_billing` — см. module.        |

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
  "in_app_promo": false,
  "in_app_billing": true,
  "browser_support": true,
  "email_support": true,
  "email_billing": true,
  "email_digest": true
}
```

</details>
