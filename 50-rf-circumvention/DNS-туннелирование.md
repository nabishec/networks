---
title: DNS-туннелирование (xDNS, DNSTT)
aliases: ["xDNS", "DNSTT", "DNS tunneling"]
type: technique
layer: application
status: partial
status_as_of: 2026-05-02
risk: low
prerequisites: ["[[DNS]]", "[[DNS — типы записей]]"]
related: ["[[Encrypted DNS — DoH-DoT]]"]
sources: [src-01, src-08]
tags: [networking, rf-circumvention, technique]
---
# DNS-туннелирование (xDNS, DNSTT)

## TL;DR
Прячут VPN-трафик в **DNS-запросы** (TXT/CNAME/NULL-records). DPI редко блокирует UDP/53 (это сломает разрешение имён). Очень медленно (~1 KB/s), не для bulk, **но пригодно для emergency**, когда ничего другого не работает. Реализации: **DNSTT** (David Fifield, 2020; используется в Snowflake), **xDNS** (русскоязычные форки).

## Какую проблему решает
В жёстких whitelist-моделях (мобильный РФ-оператор) **DNS** часто пропускается, потому что без него ничего не работает. Туннель через DNS — последняя линия обороны: даже **WebRTC** может быть закрыт, DNS — почти никогда.

## Как работает

```mermaid
flowchart LR
  Client -->|"DNS query: AAAA encoded-payload.tunnel.com"| Resolver[Local resolver]
  Resolver -->|forward| Auth[Authoritative<br/>tunnel.com]
  Auth -->|"TXT response: encoded-data"| Resolver
  Resolver --> Client
```

**Шаги:**
1. Клиент кодирует payload в base32/base64.
2. Шлёт DNS-запрос: `<base32-encoded-payload>.<tunnel-domain>`.
3. Authoritative-server (под нашим контролем) декодирует, обрабатывает.
4. Ответ кодируется в TXT-record и возвращается.

**DNSTT** улучшения:
- **Polling-based** (клиент регулярно «спрашивает» новый payload).
- Длинные ответы через несколько TXT-records.
- Шифрование payload отдельно от DNS-encoding.

## Где ломается / почему может не работать
- **Скорость:** ~1-10 KB/s teoretical. Веб-браузинг — мучение.
- **Latency:** каждый запрос — DNS round-trip.
- **DoH/DoT-detection:** некоторые провайдеры могут детектировать abnormal-pattern (много запросов к одному домену с длинными labels).
- **Whitelist-DNS** (src-05): мобильные операторы могут разрешать **только Yandex DNS (77.88.8.8)** — твой authoritative недоступен. Тогда нужно туннелировать через Yandex DNS как resolver, а authoritative — в whitelist'е.

## Минимальный пошаговый сценарий
1. Зарегистрировать домен.
2. Поднять authoritative-server (PowerDNS / NSD) на VPS.
3. Установить DNSTT-server: `dnstt-server -udp :5300 -privkey-file server.key tunnel.example.com`.
4. На клиенте: `dnstt-client -doh https://1.1.1.1/dns-query -pubkey-file server.pub tunnel.example.com 127.0.0.1:7000`.
5. Локальный 127.0.0.1:7000 — TCP-стрим к серверу через DNS.
6. Поверх — обычный SOCKS5 / shadowsocks / etc.

## Что нужно
- Свой домен.
- VPS с public IP (открытым UDP/53).
- DNSTT (https://www.bamsoftware.com/software/dnstt/).
- Стек поверх (SOCKS5).

## Связи
- **Базируется на:** [[DNS]] (механизм запросов), [[DNS — типы записей]] (TXT/AAAA/CNAME).
- **Соседи по уровню:** [[Encrypted DNS — DoH-DoT]] (DoH/DoT), [[PingTunnel — ICMP]] (другой last-resort).
- **Используется в:** Snowflake (Tor-bridge с DNSTT-fallback).
- **Противопоставляется:** обычный VPN — DNSTT в **тысячи** раз медленнее.

## Источники
- src-01, src-08.
