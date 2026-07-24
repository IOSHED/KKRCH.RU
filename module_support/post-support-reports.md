<a id="post-support-reports"></a>

### <span style="background:#42A5F5;padding:5px">POST</span> `/support/reports`

|                | Описание                                                                 |
|----------------|--------------------------------------------------------------------------|
| **Назначение** | Гостевая жалоба / фидбек без аккаунта (152-ФЗ: согласие обязательно)     |
| **Логика**     | 1. Captcha + `consent=true` + `consent_text_version`.                    |
|                | 2. Rate-limit IP/email. INSERT `support_reports`.                        |
|                | 3. Magic-link token (hash в БД) → email «жалоба принята».                |
|                | 4. Событие боту: id + kind + **subject/body** (текст в TG); без email.   |
|                | 5. Вложения не принимаются.                                              |

---

| Kind                    | Код | Описание              |
|-------------------------|-----|-----------------------|
|                         | 201 | Принято               |
| validation_error        | 400 | Поля / нет consent    |
| captcha_required_error  | 400 | Captcha               |
| too_many_requests_error | 429 | Rate limit            |
| server_error            | 500 | Ошибка                |

---

<details open>
<summary><b>Пример запроса</b></summary>

```json
{
  "email": "user@example.com",
  "kind": "abuse_link",
  "subject": "Фишинг",
  "body": "Поддельный банк по короткой ссылке.",
  "reported_url": "https://go.example/abc123",
  "consent": true,
  "consent_text_version": "support_guest_v1",
  "captcha_token": "…"
}
```

</details>

<details open>
<summary><b>Пример ответа</b></summary>

```json
{
  "id": "7c9e6679-7425-40de-944b-e07fc1f90ae7",
  "is_reviewed": false,
  "follow_up_sent_to": "u***@example.com",
  "created_at": "2026-07-21T14:00:00Z"
}
```

Email в ответе маскируется.

</details>
