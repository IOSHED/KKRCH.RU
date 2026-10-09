<a id="get-stats-utm"></a>

### <span style="background:#7CB342;padding:5px">GET</span> `/stats/{scope_id:int}/utm`

> Сводный UTM-отчёт одним запросом: качество разметки, пять срезов, рекламная
> площадка и связки `utm_source × utm_medium × utm_campaign`.
>
> **Retention:** при `coverage=raw` и `coverage=mixed` цифры считаются по
> сохранившимся строкам `link_click_events` (`window_exact=true`). При `mixed`
> клики старше cutoff в отчёт не входят. При `coverage=agg` срезы берутся из
> бессрочных карт; связки и точные доли «хотя бы одна метка» / «полная тройка»
> недоступны. [Матрица](../module_stats.md#ручки-до--после-окончания-retention).

|                | Описание                                                                 |
|----------------|--------------------------------------------------------------------------|
| **Назначение** | Заполненность UTM, top-N по каждой метке и площадке, таблица связок     |
| **Auth**       | Bearer или X-Api-Key                                                     |
| **Логика**     | 0. `StatsRpsMiddleware`                                                  |
|                | 1. Scope + entity + окно + фильтры + `coverage`                          |
|                | 2. `window_exact`: KPI по raw + шесть `GROUP BY` + связки                |
|                | 3. Иначе: срезы из `clicks_by_utm_*` / `clicks_by_ad_platform`; `combos=[]` |
|                | 4. `share` и сортировка — по `metric`                                    |
| **Параметры**  | `scope_id:int` — path                                                    |
|                | `metric:str?` — `clicks` (default) \| `spend` \| `bots`                  |
|                | `limit:int?` — top-N каждого среза; default `breakdown_top_n_default`, max `breakdown_top_n_max` |
|                | `combo_limit:int?` — top-N связок; default `ranking_limit_default`, max `ranking_limit_max` |
|                | `from`/`to` или `window`; `folder_id` / `short_id`; общие фильтры        |
| **Заголовки**  | `Cache-Control: private, max-age=<ttl>`                                  |
| **Кеш**        | `stats:utm:{scope}:{entity}:{hash}`, TTL как у breakdown (15–30 с)       |

Ключ `(none)` — клик без этой метки (`NULL` в raw). Фильтр `utm_*=(none)` **не**
матчит `NULL`: точное совпадение только с непустой строкой.

`tagged_clicks` — human-клик, у которого задана хотя бы одна из пяти меток.
`full_clicks` — заданы одновременно `utm_source`, `utm_medium` и `utm_campaign`
(content и term в «тройку» не входят). Оба поля и их `*_share` равны `null`,
когда `window_exact=false`: по независимым lifetime-картам пересечение не
восстановить.

В режиме `agg` у KPI `bots=0` и `spend="0.00"`: карты пишут только human-клики
и не хранят spend в разрезе UTM.

Фильтры при `coverage=agg` и `coverage=mixed` → `400 stats_requires_raw_error`.

---

| Ответ                         | Код | Описание                    |
|-------------------------------|-----|-----------------------------|
|                               | 200 | Отчёт построен              |
| invalid_request_params        | 400 | metric / окно / entity      |
| stats_requires_raw_error      | 400 | Фильтры при `coverage=agg` или `mixed` |
| auth_error                    | 401 | Не авторизован              |
| permission_denied_error       | 403 | Недостаточно прав           |
| api_key_other_scope           | 403 | API key привязан к другому scope |
| company_owner_not_payed_error | 402 | Подписка владельца компании не оплачена |
| scope_not_found_or_denied     | 404 | Scope недоступен            |
| folder_not_found_in_scope     | 404 | Папка недоступна            |
| short_not_found_in_scope      | 404 | Ссылка недоступна           |
| too_many_requests_error       | 429 | RPS / ban                   |
| service_unavailable           | 503 | Redis rate-limit недоступен |
| server_error                  | 500 | Внутренняя ошибка           |

---

<details open>
<summary><b>Пример запроса</b></summary>

```http
GET /api/v1/stats/10000001/utm?metric=clicks&window=30d&limit=10&combo_limit=20
Authorization: Bearer <access_token>
```

</details>

#### Пример ответа (`window_exact=true`)

```json
{
  "meta": {
    "retention_days": 90,
    "raw_available_from": "2026-07-06T00:00:00Z",
    "coverage": "raw",
    "subscription": "PERSONAL"
  },
  "entity": {
    "type": "scope",
    "scope_id": 10000001,
    "folder_id": null,
    "short_id": null
  },
  "window": {
    "preset": "30d",
    "from": "2026-09-04T00:00:00Z",
    "to": "2026-10-04T00:00:00Z"
  },
  "coverage": "raw",
  "metric": "clicks",
  "window_exact": true,
  "kpis": {
    "clicks": 1000,
    "bots": 20,
    "spend": "150.00",
    "tagged_clicks": 800,
    "tagged_share": 0.8,
    "full_clicks": 600,
    "full_share": 0.6,
    "distinct_sources": 4,
    "distinct_mediums": 3,
    "distinct_campaigns": 12,
    "distinct_contents": 6,
    "distinct_terms": 9,
    "source_fill": 0.7,
    "medium_fill": 0.65,
    "campaign_fill": 0.6,
    "content_fill": 0.2,
    "term_fill": 0.15
  },
  "dimensions": {
    "utm_source": [
      { "key": "yandex", "label": "yandex", "clicks": 700, "bots": 4, "spend": "90.00", "share": 0.7 }
    ],
    "utm_medium": [],
    "utm_campaign": [],
    "utm_content": [],
    "utm_term": [],
    "ad_platform": []
  },
  "combos": [
    {
      "utm_source": "yandex",
      "utm_medium": "cpc",
      "utm_campaign": "summer",
      "clicks": 200,
      "bots": 3,
      "spend": "40.00",
      "share": 0.2
    }
  ]
}
```

При `window_exact=false` поля `tagged_clicks`, `tagged_share`, `full_clicks`,
`full_share` равны `null`, массив `combos` пуст, `kpis.clicks` — сумма
lifetime-карты `utm_source` (включая ключ `(none)`).
