<a id="get-stats-breakdown"></a>

### <span style="background:#7CB342;padding:5px">GET</span> `/stats/{scope_id:int}/breakdown`

> Один срез (pie / bar / table) по выбранному `dimension`. Для polling-дашборда:
> отдельный запрос на каждый график **или** кэш на клиенте между тиками.
>
> **Retention:** все измерения внутри raw; вне retention — agg-maps
> (`os`/`device`/`browser`/`country`/`status_code`/`utm_*`/`target`/`ad_platform`/
> `referrer_domain`). `city` / `destination_host` вне raw → `400 stats_requires_raw_error`.
> [Матрица](../module_stats.md#ручки-до--после-окончания-retention).

|                | Описание                                                                 |
|----------------|--------------------------------------------------------------------------|
| **Назначение** | Top-N (или full) распределение метрики по измерению                      |
| **Auth**       | Bearer или X-Api-Key                                                     |
| **Логика**     | 0. `StatsRpsMiddleware`                                                  |
|                | 1. Scope + entity + окно + фильтры + `coverage`                          |
|                | 2. `GROUP BY dimension` (raw и/или agg maps)                             |
|                | 3. Сортировка по метрике desc, `limit` / merge хвоста в `(other)`        |
|                | 4. ETag / 304                                                            |
| **Параметры**  | `scope_id:int` — path                                                    |
|                | `dimension:str` — обязателен (`device`, `os`, `browser`, `country`, `city`, `status_code`, `referrer_domain`, `target`, `utm_source`, `utm_medium`, `utm_campaign`, `utm_content`, `utm_term`, `ad_platform`, `destination_host`) |
|                | `metric:str?` — `clicks` (default) \| `spend` \| `bots`                  |
|                | `limit:int?` — default `breakdown_top_n_default`, max `breakdown_top_n_max` |
|                | `include_other:bool?` — default `true` — сумма хвоста как `(other)`      |
|                | `from`/`to` или `window`; `folder_id` / `short_id`; общие фильтры        |
| **Заголовки**  | `If-None-Match:str?`                                                     |
| **Кеш**        | `stats:breakdown:{hash}`, TTL 15–30 с                                    |

`dimension=city` разрешён только при `entity=short` или `folder`, окне ≤ `7d`
**и** `coverage` с raw → иначе `400`.

`dimension=short` **нет** на этой ручке (`400 validation_error`). Рейтинг ссылок —
`GET …/ranking?by=short`.

---

| Ответ                         | Код | Описание                    |
|-------------------------------|-----|-----------------------------|
|                               | 200 | Срез построен               |
|                               | 304 | Не изменилось               |
| validation_error              | 400 | dimension / окно / entity   |
| stats_requires_raw_error      | 400 | Измерение недоступно без raw |
| auth_error                    | 401 | Не авторизован              |
| permission_denied_error       | 403 | Недостаточно прав у API key |
| api_key_scope_mismatch_error  | 403 | API key привязан к другому scope |
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
GET /api/v1/stats/10000001/breakdown?dimension=utm_campaign&metric=spend&window=30d&limit=15&ad_platform=yandex_direct
Authorization: Bearer <access_token>
```

</details>

</details>

#### Пример ответа

```json
{
  "entity": {
    "type": "scope",
    "scope_id": 10000001,
    "folder_id": null,
    "short_id": null
  },
  "window": {
    "preset": "30d",
    "from": "2026-06-16T12:00:00Z",
    "to": "2026-07-16T12:00:00Z"
  },
  "dimension": "utm_campaign",
  "metric": "spend",
  "coverage": "raw",
  "total": {
    "clicks": 22000,
    "spend": "180000.00",
    "bots": 900
  },
  "items": [
    {
      "key": "spring_promo",
      "label": "spring_promo",
      "clicks": 8000,
      "spend": "72000.00",
      "share": 0.4
    },
    {
      "key": "(none)",
      "label": "(none)",
      "clicks": 3000,
      "spend": "0.00",
      "share": 0.0
    },
    {
      "key": "(other)",
      "label": "(other)",
      "clicks": 1500,
      "spend": "9000.00",
      "share": 0.05
    }
  ]
}
```

`share` — доля выбранной `metric` от `total` (для `spend` — от total.spend).

Для `dimension=target`:

```json
{
  "key": "42",
  "label": "https://www.example.com/landing-a",
  "clicks": 1200,
  "spend": "9600.00",
  "share": 0.31
}
```

`label` — актуальный `short_targets.url` (если target удалён → `"deleted"`).

<details>
<summary><b>До / после retention</b></summary>

| Поле / возможность | До retention (`coverage=raw`) | После retention (`coverage=agg`) |
|--------------------|-------------------------------|----------------------------------|
| HTTP-код | 200 | 200 или **400** `stats_requires_raw_error` |
| `coverage` | `raw` | `agg` |
| `dimension` os / device / browser / country / status_code | ✅ в **окне** запроса | ⚠️ **lifetime** maps agg (окно **не** фильтрует) |
| `dimension` utm_* / target / ad_platform / referrer_domain | ✅ в окне | ⚠️ lifetime maps |
| `dimension` city | ✅ (entity short/folder, окно ≤ 7d) | ❌ **400** |
| `dimension` destination_host | ✅ | ❌ **400** |
| `dimension` short | ❌ нет на breakdown (`validation_error`); см. `ranking?by=short` | ❌ |
| `metric` clicks / spend / bots | ✅ | ✅ clicks/spend; `bots` = 0 |
| `total`, `items`, `share` | ✅ по окну | ⚠️ по lifetime (share от lifetime total) |
| Фильтры (`utm_*`, `target_id`, …) | ✅ | ❌ **400** |
| `mixed` (окно пересекает cutoff) | ✅ raw за всё окно* | — (только raw или agg) |

\*При `coverage=mixed` breakdown всё ещё читает raw за полное окно (если raw-строки остались).

</details>
