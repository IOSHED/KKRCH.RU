<a id="post-support-admin-users-id-notifications"></a>

### <span style="background:#42A5F5;padding:5px">POST</span> `/support/admin/users/{user_id:uuid}/notifications`

|                | Описание                                                         |
|----------------|------------------------------------------------------------------|
| **Назначение** | Точечное in-app уведомление одному пользователю (без campaign) |
| **Логика**     | 1. Service token. 2. User существует. 3. INSERT `user_notifications`. |
|                | 4. `kind` default `custom`; prefs не блокируют.                  |

---

| Kind                 | Код | Описание        |
|----------------------|-----|-----------------|
|                      | 201 | Создано         |
| validation_error     | 400 | title/body      |
| auth_error           | 401 | Нет token       |
| user_not_found_error | 404 | Нет user        |
| server_error         | 500 | Ошибка          |

---

<details open>
<summary><b>Пример</b></summary>

```json
{
  "title": "Проверьте email",
  "body": "Мы обновили условия сервиса.",
  "kind": "custom"
}
```

</details>
