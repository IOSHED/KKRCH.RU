<a id="patch-support-admin-reports"></a>

### <span style="background:#FB8C00;padding:5px">PATCH</span> `/support/admin/reports/{report_id:uuid}`

|                | Описание                                                              |
|----------------|-----------------------------------------------------------------------|
| **Назначение** | Оператор помечает жалобу рассмотренной; опционально действие по short |
| **Логика**     | 1. Service token. 2. `is_reviewed`, `admin_action`.                   |
|                | 3. При `short_deactivated` + известном `short_id` — deactivate/archive.|
|                | 4. `support_audit_log`.                                               |

---

| Kind                   | Код | Описание     |
|------------------------|-----|--------------|
|                        | 200 | Обновлено    |
| auth_error             | 401 | Нет token    |
| report_not_found_error | 404 | Нет записи   |
| server_error           | 500 | Ошибка       |

---

<details open>
<summary><b>Пример запроса</b></summary>

```json
{
  "is_reviewed": true,
  "admin_action": "short_deactivated"
}
```

</details>
