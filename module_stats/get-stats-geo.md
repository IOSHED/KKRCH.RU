<a id="get-stats-geo"></a>

### <span style="background:#7CB342;padding:5px">GET</span> `/stats/{scope_id:int}/geo`

> Geo-карта кликов для вкладки «Статистика → Графики»: choropleth мира / субъектов РФ
> или hex-теплокарта по координатам.
>
> **Retention:** `countries` / `regions` работают из agg (`clicks_by_country` /
> `clicks_by_region`) после retention; `hex` требует raw (`geo_lat`/`geo_lon`) → иначе
> `400 stats_requires_raw_error`.

|                | Описание                                                                    |
|----------------|-----------------------------------------------------------------------------|
| **Назначение** | Агрегаты для картографической визуализации                                  |
| **Auth**       | Bearer или X-Api-Key                                                        |
| **Логика**     | 0. `StatsRpsMiddleware`                                                     |
|                | 1. Scope + entity + окно + `coverage`                                       |
|                | 2. `mode=countries` → GROUP BY `geo_country` (или agg map)                  |
|                | 3. `mode=regions&country=RU` → GROUP BY `geo_region` (`RU-NIZ`, …)          |
|                | 4. `mode=hex` → grid `floor(lat/res)`, `floor(lon/res)`                     |
| **Параметры**  | `mode:str` — `countries` \| `regions` \| `hex`                              |
|                | `country:str?` — для regions обязателен `RU` (v1)                           |
|                | `metric:str?` — `clicks` (default) \| `bots` (`spend` → 400)                |
|                | `resolution:float?` — только hex, градусы, default `0.5`, clamp `[0.05, 5]` |
|                | `limit:int?` — default 200                                                  |
|                | `from`/`to` или `window`; `folder_id` / `short_id`; фильтры UTM/device/…    |
| **Кеш**        | `stats:geo:{hash}`, TTL как у breakdown                                     |

Ключ региона — `{CC}-{SUB}` из GeoLite2 `subdivision_1_iso_code` (см. enrichment /
`import_geolite2.sh`).

---

| Ответ                    | Код | Описание                       |
|--------------------------|-----|--------------------------------|
|                          | 200 | Карта построена                |
| validation_error         | 400 | mode / country / metric / окно |
| stats_requires_raw_error | 400 | hex вне raw                    |
| auth_error               | 401 | Не авторизован                 |
| permission_denied_error  | 403 | Недостаточно прав API key      |
| scope_not_found_error    | 404 | Scope недоступен               |
| too_many_requests_error  | 429 | RPS / ban                      |
| server_error             | 500 | Внутренняя ошибка              |

---

<details open>
<summary><b>Пример запроса</b></summary>

```http
GET /api/v1/stats/10000001/geo?mode=regions&country=RU&window=7d
Authorization: Bearer <access_token>
```

</details>

#### Пример ответа (regions)

```json
{
  "mode": "regions",
  "metric": "clicks",
  "coverage": "raw",
  "country": "RU",
  "total": {
    "clicks": 1200,
    "bots": 40,
    "unknown": 15
  },
  "items": [
    {
      "key": "RU-NIZ",
      "label": "RU-NIZ",
      "value": 320,
      "share": 0.27
    },
    {
      "key": "RU-MOW",
      "label": "RU-MOW",
      "value": 210,
      "share": 0.18
    }
  ],
  "cells": null
}
```

#### Пример ответа (hex)

```json
{
  "mode": "hex",
  "metric": "clicks",
  "coverage": "raw",
  "resolution": 0.5,
  "total": {
    "clicks": 900,
    "bots": 12,
    "unknown": 0
  },
  "items": null,
  "cells": [
    {
      "lat": 55.75,
      "lon": 37.62,
      "value": 88
    }
  ]
}
```
