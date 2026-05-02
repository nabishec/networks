---
title: Loyalsoldier geo-files
aliases: ["geosite.dat", "geoip.dat", "v2ray-rules-dat"]
type: tool
layer: application
status: working
status_as_of: 2026-05-02
risk: low
prerequisites: ["[[BGP]]", "[[DNS]]"]
related: ["[[Split routing]]", "[[Xray-core]]", "[[Sing-box]]"]
sources: [src-02, src-03, src-08]
tags: [networking, rf-circumvention, tool]
---
# Loyalsoldier geo-files (geosite.dat / geoip.dat)

## TL;DR
**Готовые dataset'ы доменов и IP-префиксов** для маршрутизации в Xray/Sing-box. Заменяют сотни ручных правил в `routing.rules`. Поддерживаются проектом **v2ray-rules-dat** (GitHub: Loyalsoldier/v2ray-rules-dat); обновляются ежедневно с помощью CDN-сборок Cloudflare/jsDelivr.

## Что внутри

### `geosite.dat` (домены, ~50 MB)
- `geosite:cn` — китайские домены.
- `geosite:ru` — российские домены (`*.yandex.ru`, `*.gosuslugi.ru`, ...).
- `geosite:category-ads-all` — реклама.
- `geosite:google`, `geosite:youtube`, `geosite:netflix` — конкретные сервисы.
- `geosite:gfw` — заблокированные в Китае.

### `geoip.dat` (IP-префиксы, ~5 MB)
- `geoip:cn` — все CN AS.
- `geoip:ru` — все RU AS (более 5000 префиксов).
- `geoip:private` — RFC 1918.
- `geoip:google`, `geoip:cloudflare` — конкретные провайдеры.

## Какую проблему решает
Без geo-файлов **split routing** превращается в ручное ведение списка из тысяч правил. Loyalsoldier:
- Агрегирует RIR-data (RIPE, APNIC) + сообщество-curated списки.
- Релизы ежедневно (`*.dat` бинарный формат для быстрого парсинга).
- Cross-platform (Xray, Sing-box, Clash используют один и тот же формат).

## Как использовать

### Установка
```bash
# Скачать в Xray data-dir:
cd /usr/local/share/xray
wget https://github.com/Loyalsoldier/v2ray-rules-dat/releases/latest/download/geosite.dat
wget https://github.com/Loyalsoldier/v2ray-rules-dat/releases/latest/download/geoip.dat
```

### Применение в Xray-config
```json
"routing": {
  "rules": [
    { "type": "field", "domain": ["geosite:ru"], "outboundTag": "direct" },
    { "type": "field", "ip": ["geoip:ru", "geoip:private"], "outboundTag": "direct" },
    { "type": "field", "domain": ["geosite:category-ads-all"], "outboundTag": "block" },
    { "type": "field", "outboundTag": "proxy" }
  ]
}
```

### Auto-update
Cron-job обновляет `.dat`-файлы раз в неделю. Перезапуск Xray не нужен (горячая загрузка).

## Где ломается / лимиты
- **Список не идеален:** какой-то РФ-сервис может оказаться вне `geosite:ru` (например, региональный) → его трафик пойдёт через VPN.
- **Размер:** на embedded-роутерах (256 MB RAM) `geosite.dat` (~50 MB) занимает заметную долю.
- **Альтернативы:** v2fly/domain-list-community — upstream community-version, более «нейтральный»; китайские — chocolate4u.

## Связи
- **Базируется на:** [[BGP]] (RIR-данные → IP-префиксы), [[DNS]] (домены).
- **Используется в:** [[Split routing]] (главный потребитель), [[Xray-core]], [[Sing-box]].
- **Соседи по уровню:** v2fly/domain-list-community, chocolate4u/ircf-space (ирано-специфичный).

## Источники
- src-02, src-03, src-08.
