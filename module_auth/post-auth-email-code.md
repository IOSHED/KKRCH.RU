<a id="post-auth-email-code"></a>

### <span style="background:#42A5F5;padding:5px">POST</span> `/auth/email/code`

|                | Описание |
|----------------|----------|
| **Назначение** | Отправить 6-значный код на email |
| **Логика**     | 1. Email нормализуется: trim + нижний регистр. |
|                | 2. Повтор раньше `resend_after` — `200` и письмо не уходит. |
|                | 3. Код (хеш) кладётся в Redis, письмо уходит сразу через SMTP или file-outbox. |
|                | 4. Ответ не говорит, есть ли аккаунт. |

---

| Kind             | Код | Описание |
|------------------|-----|----------|
|                  | 200 | Запрос принят (`{"ok": true}`) |
| validation_error | 400 | Некорректный email |
| server_error     | 500 | Почта не отправилась |

---

<details open>
<summary><b>Пример запроса</b></summary>

```json
{
  "email": "user@example.com"
}
```

</details>

<details open>
<summary><b>Пример ответа</b></summary>

```json
{ "ok": true }
```

</details>
