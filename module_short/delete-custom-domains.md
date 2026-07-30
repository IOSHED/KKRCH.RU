<a id="delete-custom-domains"></a>

### <span style="background:#EF5350;padding:5px">DELETE</span> `/custom-domains`

|                      | Описание                                                                                                                |
|----------------------|-------------------------------------------------------------------------------------------------------------------------|
| **Назначение**       | Мягко удаляет собственный домен и деактивирует ссылки на нём                                                            |
| **Auth**             | Bearer или X-Api-Key                                                                                                    |
| **Логика**           | 1. Проверяет принадлежность домена пользователю (JOIN со `scopes`)                                                      |
|                      | 2. `deleted_at = now()`, `status = deleted`                                                                             |
|                      | 3. Деактивирует все shorts с `custom_domain = domain`: `is_active = false`, `inactive_reason = 'custom_domain_deleted'` |
|                      | &nbsp;&nbsp;&nbsp;Ссылки, уже деактивированные вручную, **не** перезаписываются                                         |
|                      | 4. Инвалидирует resolve/availability кеш для домена                                                                     |
|                      | 5. Домен можно снова зарегистрировать через `POST /custom-domains` (новый `verification_token`)                         |
| **Параметры**        | `domain:str` — FQDN в body                                                                                              |
| **Инвалидация кеша** | `INCR custom_domains:list:{owner}:v`; `DEL short:resolve:{custom_domain}:*`; availability keys                          |

---

| Ответ                        | Код | Описание                         |
|------------------------------|-----|----------------------------------|
| success                      | 204 | Домен удалён                     |
| auth_error                   | 401 | Не авторизован                   |
| permission_denied_error      | 403 | Недостаточно прав у API key      |
| api_key_scope_mismatch_error | 403 | API key привязан к другому scope |
| not_found                    | 404 | Домен не найден                  |
| server_error                 | 500 | Внутренняя ошибка сервера        |

---

<details open>
<summary><b>Пример запроса</b></summary>

```json
{
  "domain": "go.company.ru"
}
```

</details>
