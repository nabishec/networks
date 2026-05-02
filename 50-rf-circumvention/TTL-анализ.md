---
title: TTL-анализ (геолокация ТСПУ)
aliases: ["TTL-анализ", "TTL fingerprint", "Hop-count fingerprint"]
type: concept
layer: network
status: working
status_as_of: 2026-05-02
risk: low
prerequisites: ["[[ТСПУ]]", "[[ICMP]]", "[[IPv4]]"]
related: ["[[PB4 — диагностика whitelist]]", "[[NAT]]", "[[Active probing]]"]
sources: [src-01, src-05]
tags: [networking, rf-circumvention, concept]
---
# TTL-анализ — геолокация ТСПУ через hop-count

## TL;DR
**Метод определения, на каком hop'е сети сидит ТСПУ-устройство.** IP-пакет несёт `TTL` (Time To Live) — счётчик, уменьшаемый каждым роутером на 1. Зная **исходный** TTL разных ОС (Linux=64, Windows=128, BSD=64, mobile-iOS=64) и **приходящий** TTL ответа от ТСПУ при блокировке (RST/redirect-ответ), можно посчитать hop-count и сопоставить с traceroute, чтобы локализовать узел DPI на 2-м или 3-м хопе РФ-оператора.

В src-01 описан как один из 6 способов выявления механики whitelist'а: «ТСПУ работает на 2-м хопе и генерирует RST для заблокированных IP».

## Какую проблему решает
- Подтвердить, что **ваш конкретный** оператор использует ТСПУ (не региональный CDN или просто медленный канал).
- Локализовать **точку дропа** в сети — критично для выбора техники обхода (на L3 в опере? на bgp-стыке?).
- Распознать **fake-RST**: если RST «приходит» от destination (TTL соответствует hop-count'у destination), это легитимный сброс. Если TTL **выше** ожидаемого (мало хопов до отправителя) — RST спуфит ТСПУ на 2-3 хопе.

## Как работает

### Метрика
1. **Стандартные initial-TTL по ОС:**
   - Linux / Android: 64.
   - macOS / iOS / *BSD: 64.
   - Windows 10/11: 128.
   - Cisco IOS: 255.
2. **Hop-count = initial − received.**
3. Для destination в Германии (Hetzner): обычно 18-22 хопа из РФ.
4. Если получаем RST с TTL=62 (т.е. hop-count=2 от Linux-источника) — это RST не от destination, а от **узла на 2-м хопе** (т.е. ТСПУ).

### Демонстрация
```bash
# Уязвимый запрос:
curl -v https://blocked-site.com 2>&1 | head -20

# Параллельно: tcpdump для RST-анализа:
sudo tcpdump -ni any -v 'tcp[tcpflags] & tcp-rst != 0' -c 1
# → IP (tos 0x0, ttl 62, id 0, ...) X.X.X.X.443 > local.54321: Flags [R]
#                       ^^^^^^ → hop-count = 64 − 62 = 2 → ТСПУ на 2-м хопе
```

### Сравнение с traceroute
```bash
traceroute -n blocked-site.com
# 1   192.168.0.1     1 ms          (роутер дома)
# 2   100.64.0.1      8 ms          ← CGNAT-узел оператора + ТСПУ
# 3   *  *  *                       (asymmetric routing / drop)
# 4   ...
```
Узел `100.64.x.x` или `198.18.x.x` на 2-м хопе → почти наверняка ТСПУ-карта (см. src-05).

## Применение

### 1. Идентификация ТСПУ
Если RST на blocked-site приходит с TTL = `init_TTL - 2`, а до самого destination минимум 15-20 хопов — это **точно** не RST от destination, а спуфинг от ТСПУ.

### 2. Понимание whitelist-режима
Если ТСПУ **тихо дропает** (без RST) — TTL-проба не помогает. Только косвенно через [[Session freezing|16-КБ-отсечку]] или DNS-spoofing-тесты.

### 3. Обход: TTL-injection
Некоторые DPI-bypass-tools ([[Zapret GUI]], GoodbyeDPI) умеют отправлять «фальшивые» пакеты с **низким** TTL: пакет дойдёт до ТСПУ, активирует его detection-логику, но не дойдёт до destination. Реальный пакет с нормальным TTL идёт следом → ТСПУ уже «решил», что это шум, и пропускает следующий поток. Эффективно против старых DPI-классов, не против современного whitelist-режима.

## Связи
- **Базируется на:** [[ICMP]] (TTL-механика), [[IPv4]] (поле TTL), [[Three-way handshake]] (когда RST приходит).
- **Используется в:** [[PB4 — диагностика whitelist]] (один из шагов), [[Zapret GUI]] (TTL-injection bypass), [[Active probing]] (косвенно).
- **Соседние понятия:** **CGNAT** ([[NAT]]) и узлы `100.64.x.x` / `198.18.x.x` как маркер ТСПУ-карты.
- **Где не работает:** ТСПУ в whitelist-режиме часто **не отвечает RST**, а тихо дропает пакеты → TTL-анализ бесполезен.

## Источники / Дальше читать
- [src-01 — Это всё что вам надо знать о белых списках](https://habr.com/ru/articles/1027276/) — TTL-анализ как один из 6 способов выявления механики.
- [src-05 — Reality in Whitelists](https://habr.com/ru/articles/990190/) — узлы CGNAT/ТСПУ на 2-3 хопе мобильных операторов.
- Habr: [Как работают ТСПУ и DPI](https://habr.com/ru/articles/992232/) — общая механика DPI.
- Habr: [Какие адреса мы видим в traceroute](https://habr.com/ru/articles/329244/) — про MPLS/TTL и видимость хопов.
- Habr: [Traceroute: про умение читать вывод](https://habr.com/ru/articles/129627/) — диагностика hop-count'ов.
