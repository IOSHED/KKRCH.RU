<a id="post-transfer-imports-preflight"></a>

### <span style="background:#42A5F5;padding:5px">POST</span> `/transfer/{scope_id:int}/imports/preflight`

|                | Описание                                                                                  |
|----------------|-------------------------------------------------------------------------------------------|
| **Назначение** | Синхронная проверка import **до** создания job и тяжёлой загрузки файла                   |
| **Логика**     | 1. Bearer + доступ к scope.                                                               |
|                | 2. Проверяет `subscription_plans.transfer_enabled` (FREE → 402).                          |
|                | 3. Stream-scan файла (без записи в БД): подсчёт shorts, subdomain, raw dates.               |
|                | 4. Сверка с лимитами `max_shorts`, subdomain, retention.                                  |
|                | 5. При успехе → `200` + сводка; при нарушении → **402/400** + `details`, job не создаётся. |
| **Параметры**  | `scope_id:int` — path                                                                     |

> **Preflight optional:** клиент может не вызывать эту ручку — те же проверки
> выполняются inline на [`POST …/imports`](post-transfer-imports.md). Preflight
> экономит upload; безопасность не ослабляется.

---

| Kind                        | Код | Описание                                              |
|-----------------------------|-----|-------------------------------------------------------|
|                             | 200 | Preflight пройден, import разрешён                    |
| stats_retention_error       | 400 | Raw-клики вне retention (`ignore_retention_limit=false`) |
| validation_error            | 400 | metadata / формат файла                               |
| auth_error                  | 401 | Не авторизован                                        |
| scope_not_found_error       | 404 | Scope недоступен                                      |
| short_name_not_payed_error  | 402 | Превышен `max_shorts`                                 |
| subdomain_not_payed_error   | 402 | Subdomain из файла не существуют / лимит subdomain    |
| transfer_not_payed_error    | 402 | `transfer_enabled=false` (тариф FREE)                 |
| payload_too_large_error     | 413 | Файл > лимита upload                                  |
| server_error                | 500 | Внутренняя ошибка                                     |

---

<details open>
<summary><b>Пример запроса</b></summary>

Тело — идентично `POST …/imports` (multipart: `metadata` + `file`).

```json
{
  "adapter": "linkly_links",
  "format": "csv",
  "ignore_retention_limit": false,
  "dry_run": false
}
```

</details>

<details open>
<summary><b>Пример ответа 200</b></summary>

```json
{
  "ok": true,
  "subscription": "PERSONAL",
  "limits": {
    "max_shorts": 300,
    "max_subdomains": 0,
    "stats_click_retention_days": 90
  },
  "import_summary": {
    "would_create_shorts": 120,
    "current_shorts": 45,
    "remaining_short_slots": 255,
    "unique_subdomains": [],
    "missing_subdomains": [],
    "raw_events_total": 158000,
    "raw_events_outside_retention": 0
  }
}
```

При нарушении лимитов вместо `200` возвращается **402/400** (см. примеры ниже).

</details>

<details open>
<summary><b>402 — transfer_not_payed_error (FREE)</b></summary>

```json
{
  "kind": "transfer_not_payed_error",
  "reason": "Import/export доступен на тарифах PERSONAL и выше",
  "details": {
    "subscription": "FREE",
    "transfer_enabled": false,
    "upgrade_plan": "PERSONAL"
  }
}
```

Проверка выполняется **до** scan файла — минимальная нагрузка даже при
скомпрометированном токене FREE-пользователя.

</details>

<details open>
<summary><b>402 — short_name_not_payed_error</b></summary>

```json
{
  "kind": "short_name_not_payed_error",
  "reason": "Import добавит 8420 ссылок при лимите FREE = 10",
  "details": {
    "subscription": "FREE",
    "current_count": 8,
    "plan_limit": 10,
    "import_would_add": 8420,
    "over_limit_by": 8418,
    "remaining_slots": 2
  }
}
```

</details>

<details open>
<summary><b>402 — subdomain_not_payed_error</b></summary>

```json
{
  "kind": "subdomain_not_payed_error",
  "reason": "В файле указаны subdomain, которых нет у аккаунта",
  "details": {
    "missing_subdomains": ["brand", "promo-campaign"],
    "referenced_in_rows": 1240,
    "plan_max_subdomains": 0,
    "owned_subdomains": []
  }
}
```

Фронт показывает CTA «Создать subdomain» (как при `subdomain_not_payed_error`
в create short).

</details>

<details open>
<summary><b>400 — stats_retention_error</b></summary>

```json
{
  "kind": "stats_retention_error",
  "reason": "842000 raw-кликов старше 30 дней (лимит retention FREE)",
  "details": {
    "retention_days": 30,
    "cutoff_at": "2026-06-21T00:00:00Z",
    "oldest_event_at": "2024-03-01T12:00:00Z",
    "events_outside_retention": 842000,
    "events_within_retention": 158000,
    "ignore_available": true
  }
}
```

Повторите import с `"ignore_retention_limit": true` — raw вне окна будет
**пропущен**, agg обновится (см. [Import статистики](../module_transfer.md#import-статистики-raw--agg)).

</details>

<details>
<summary><b>Поле details</b></summary>

Transfer-ручки расширяют стандартный [`ErrorBody`](../module_auth.md) optional
полем `details: object` для машинной обработки на фронте. Остальные модули
API по-прежнему `{ kind, reason }` only.

</details>
