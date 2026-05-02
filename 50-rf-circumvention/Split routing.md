---
title: Split routing
aliases: ["Split tunneling", "Split routing", "Раздельная маршрутизация"]
type: technique
layer: network
status: working
status_as_of: 2026-05-02
risk: low
prerequisites: ["[[Маршрутизатор]]", "[[Иерархическая маршрутизация]]"]
related: ["[[vnext-цепочка]]"]
sources: [src-02, src-03, src-08]
tags: [networking, rf-circumvention, technique]
---
# Split routing (split tunneling)

## TL;DR
Маршрутизация **только нужного** трафика через VPN; остальное — DIRECT (через локального провайдера). RU-домены (Yandex, Mail.ru, Госуслуги) идут напрямую — быстрее, не нагружают VPN, не вызывают подозрений; зарубежные — через туннель. Реализуется через **geo-файлы** (`geosite.dat`, `geoip.dat`), правила в Xray/Sing-box.

## Какую проблему решает
- **Performance:** RU-сайты быстрее без VPN-крюка.
- **Whitelist friendly:** часть трафика идёт по «легитимным» путям, не через VPN-сервер → меньше следов.
- **Privacy сегрегация:** банковские/госуслуги — DIRECT (не уходят в зарубежный VPS); прочее — VPN.

## Как работает

**Xray routing-block:**
```json
"routing": {
  "rules": [
    {
      "type": "field",
      "domain": ["geosite:cn", "geosite:category-ru"],
      "outboundTag": "direct"
    },
    {
      "type": "field",
      "ip": ["geoip:ru"],
      "outboundTag": "direct"
    },
    {
      "type": "field",
      "outboundTag": "proxy"  // default — всё остальное в VPN
    }
  ]
}
```

**Geo-файлы:**
- `geosite.dat` от Loyalsoldier — список доменов по категориям (`category-ru`, `cn`, `private`, `apple-domains`, `geolocation-cn`, …).
- `geoip.dat` — IP-prefixes по странам (geoip:ru, geoip:us, …).
- Обновляется регулярно через GitHub releases.

**Mihomo / Sing-box** — аналогичные правила.

## Где ломается / почему может не работать
- **Geo-файлы устаревают** — если сайт сменил CDN, может быть routed неправильно.
- **DNS-leak:** если DNS-запрос идёт через DIRECT, провайдер видит, что ты резолвишь зарубежный домен → splitting только трафика, не DNS, легко обнаружимо.
- **Some apps игнорируют** system proxy/routing (GAMEs с anti-cheat, mobile native apps) → требуется VPN-mode.
- **«Утечка» в DIRECT:** если правило неправильное, частный трафик может уйти провайдеру в открытом виде.

## Минимальный пошаговый сценарий
1. Скачать `geosite.dat` и `geoip.dat` от Loyalsoldier в директорию Xray (`/usr/local/share/xray/`).
2. Добавить routing-rules в Xray-конфиг (см. выше).
3. Перезапустить Xray.
4. Проверить через `curl --interface ...` или `traceroute` к разным доменам.

## Что нужно
- Xray ≥ v25 или Sing-box ≥ v1.x или Mihomo.
- Geo-файлы (downloaded).
- Клиент, поддерживающий geo-routing (Hiddify, Nekoray).

## Связи
- **Базируется на:** [[Маршрутизатор]] (концепт routing), [[Иерархическая маршрутизация]].
- **Используется в:** [[vnext-цепочка]], [[PB2 — vnext-цепочка через РФ-мост]], [[PB3 — 4-уровневая архитектура за 265₽]].
- **Соседи по уровню:** **DNS-leak protection** (отдельная категория).
- **Противопоставляется:** «всё через VPN» — медленнее, более подозрительно.

## Источники
- src-02, src-03, src-08.
