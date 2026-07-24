<a id="delete-subdomains"></a>

### <span style="background:#EF5350;padding:5px">DELETE</span> `/subdomains/{subdomain:str}`

|                      | Описание                                                                                           |
|----------------------|----------------------------------------------------------------------------------------------------|
| **Назначение**       | Мягко удаляет subdomain и деактивирует его ссылки                                                  |
| **Логика**           | 1. Проверяет принадлежность subdomain текущему пользователю (JOIN со `scopes` по `owner_user_id`)  |
|                      | 2. Устанавливает `subdomains.deleted_at = now()` (soft-delete, физически запись остаётся)          |
|                      | 3. Деактивирует все ссылки поддомена: `is_active = false`, `inactive_reason = 'subdomain_deleted'` |
|                      | &nbsp;&nbsp;&nbsp;Ссылки, уже деактивированные пользователем вручную, не перезаписываются          |
|                      | 4. Инвалидирует Redis-кеш availability и resolve для данного поддомена                             |
|                      | Поддомен можно восстановить через `POST /subdomains` с тем же именем                               |
| **Параметры**        | `subdomain:str` - имя поддомена                                                                    |
| **Инвалидация кеша** | Очистка `shorts:availability:{subdomain}:*` и `short:resolve:{subdomain}:*`                        |

---

| Ответ        | Код | Описание                  |
|--------------|-----|---------------------------|
| success      | 204 | Subdomain удален          |
| auth_error   | 401 | Не авторизован            |
| not_found    | 404 | Subdomain не найден       |
| server_error | 500 | Внутренняя ошибка сервера |

---
