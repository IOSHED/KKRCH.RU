<a id="get-stats-ranking"></a>

### <span style="background:#7CB342;padding:5px">GET</span> `/stats/{scope_id:int}/ranking`

> Таблица «топов» внутри entity: ссылки, targets, UTM-кампании, referrers.
> Удобна для scope/folder overview tables и polling раз в 30–60 с.
>
> **Retention:** полный `by` внутри raw. При `coverage=mixed` ranking **склеивает**
> `stats_by_day` (дни &lt; cutoff) + raw (дни ≥ cutoff) — см. `window_merge`.
> [Матрица](../module_stats.md#ручки-до--после-окончания-retention).

|                | Описание                                                                 |
|----------------|--------------------------------------------------------------------------|
| **Назначение** | Упорядоченный список сущностей с метриками за окно                       |
| **Логика**     | 0. `StatsRpsMiddleware`                                                  |
|                | 1. Scope + entity (`short_id` запрещён для `by=short`)                   |
|                | 2. Выбор источника (raw / agg / mixed) по окну и `by`                    |
|                | 3. `by=short` + `mixed`: per-short merge `stats_by_day` + raw            |
|                | 4. `by=dimension` + `mixed`: merge day-maps измерения + raw-сегмент      |
|                | 5. `coverage=agg` + dimension: lifetime maps (`window_merge=agg_lifetime`) |
|                | 6. Агрегация + JOIN метаданных; сортировка `order_by` + `limit`          |
| **Параметры**  | `scope_id:int` — path                                                    |
|                | `by:str` — `short` \| `target` \| `utm_campaign` \| `utm_source` \| `referrer_domain` \| `ad_platform` \| `country` |
|                | `order_by:str?` — `clicks` (default) \| `spend` \| `bots`                |
|                | `limit:int?` — default 20, max 100                                       |
|                | `from`/`to` или `window`; `folder_id` / `short_id`; общие фильтры        |
| **Кеш**        | `stats:ranking:{hash}`, TTL 30–60 с                                      |

Допустимые комбинации `by` × entity:

| `by` | scope | folder | short |
|------|-------|--------|-------|
| `short` | ✅ | ✅ | ❌ |
| `target` | ✅ | ✅ | ✅ |
| `utm_*` / `ad_platform` / `referrer_domain` / `country` | ✅ | ✅ | ✅ |

### `window_merge` — подсказка для frontend

| Значение | Когда | Что показывать пользователю |
|----------|-------|-----------------------------|
| *(нет)* | `coverage=raw` | Полное окно из raw |
| `full` | `coverage=mixed` | Лидеры за **всё** окно; ссылки с кликами только до retention и только после — **обе** попадают в таблицу |
| `agg_lifetime` | `coverage=agg` + `by≠short` | Lifetime totals; **не** фильтруется по `window` — показать badge «за всё время» |

При `coverage=agg` + `by=short` метрики считаются из `stats_by_day` **внутри окна** (rolling 30d).

---

| Ответ                         | Код | Описание                    |
|-------------------------------|-----|-----------------------------|
|                               | 200 | Ranking собран              |
| validation_error              | 400 | by / entity / окно          |
| stats_requires_raw_error      | 400 | `by` недоступен без raw/agg map |
| auth_error                    | 401 | Не авторизован              |
| scope_not_found_error         | 404 | Scope недоступен            |
| folder_not_found_error        | 404 | Папка недоступна            |
| short_not_found_error         | 404 | Ссылка недоступна           |
| too_many_requests_error       | 429 | RPS / ban                   |
| service_unavailable           | 503 | Redis rate-limit недоступен |
| server_error                  | 500 | Внутренняя ошибка           |

---

<details open>
<summary><b>Пример запроса</b></summary>

```http
GET /api/v1/stats/10000001/ranking?by=short&folder_id=abc12345-e89b-12d3-a456-426614174000&window=7d&order_by=spend&limit=10
Authorization: Bearer <access_token>
```

</details>

#### Пример ответа

```json
{
  "entity": {
    "type": "folder",
    "scope_id": 10000001,
    "folder_id": "abc12345-e89b-12d3-a456-426614174000",
    "short_id": null
  },
  "window": {
    "preset": "7d",
    "from": "2026-07-09T12:00:00Z",
    "to": "2026-07-16T12:00:00Z"
  },
  "by": "short",
  "order_by": "spend",
  "coverage": "raw",
  "items": [
    {
      "rank": 1,
      "short_id": 10000042,
      "short_name": "summer-sale",
      "folder_id": "abc12345-e89b-12d3-a456-426614174000",
      "clicks": 520,
      "bots": 14,
      "spend": "4100.00",
      "avg_ttfb_ms": 10.2
    },
    {
      "rank": 2,
      "short_id": 10000055,
      "short_name": "blog-post",
      "folder_id": "abc12345-e89b-12d3-a456-426614174000",
      "clicks": 300,
      "bots": 40,
      "spend": "0.00",
      "avg_ttfb_ms": 9.1
    }
  ]
}
```

Пример `coverage=mixed` (окно пересекает retention):

```json
{
  "coverage": "mixed",
  "window_merge": "full",
  "items": [
    {
      "rank": 1,
      "short_id": 10000042,
      "short_name": "old-campaign",
      "clicks": 180,
      "bots": 3,
      "spend": "900.00",
      "avg_ttfb_ms": 11.0
    },
    {
      "rank": 2,
      "short_id": 10000099,
      "short_name": "fresh-link",
      "clicks": 150,
      "bots": 2,
      "spend": "750.00",
      "avg_ttfb_ms": 9.5
    }
  ]
}
```

Ссылка `old-campaign` могла иметь клики **только** до cutoff (из `stats_by_day`),
`fresh-link` — **только** после (из raw); обе корректно сортируются в одном топе.

Пример `by=target` (entity = short):

```json
{
  "by": "target",
  "items": [
    {
      "rank": 1,
      "target_id": 1,
      "url": "https://www.example.com/a",
      "weight": 70,
      "clicks": 364,
      "spend": "2870.00",
      "is_active": true
    }
  ]
}
```

<details>
<summary><b>До / после retention</b></summary>

| Поле / возможность | До retention (`coverage=raw`) | После / mixed (`coverage=agg` / `mixed`) |
|--------------------|-------------------------------|------------------------------------------|
| HTTP-код | 200 | 200 или **400** `stats_requires_raw_error` |
| `coverage` | `raw` | `agg` или `mixed` |
| `window_merge` | — | `full` (mixed) / `agg_lifetime` (agg + dimension) |
| **`by=short`** — clicks/spend/bots в окне | ✅ raw | ✅ **`full` merge** per short |
| **`by=short`** — `avg_ttfb_ms` | ✅ | ✅ из `stats_by_day.sum_ttfb_ms` / `ttfb_samples` + raw |
| **`by=target\|utm_*\|country\|…`** в окне | ✅ raw | ✅ **`full` merge** (day-maps + raw) при `mixed` |
| **`by=target\|utm_*\|…`** при `coverage=agg` | — | ⚠️ **lifetime** maps (`agg_lifetime`) |
| Фильтры | ✅ | ❌ **400** при `coverage=agg` или `mixed` |
| Ссылки «только до retention» в топе | ✅ | ✅ при `mixed` + `window_merge=full` |

</details>
