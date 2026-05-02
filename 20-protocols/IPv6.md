---
title: IPv6
aliases: ["Internet Protocol v6", "IP v6"]
type: protocol
layer: network
chapter: 5
difficulty: basic
prerequisites: ["[[IPv4]]"]
related: ["[[IP-адресация и CIDR]]", "[[NAT]]", "[[ICMP]]"]
tags: [networking, ch05]
---
# IPv6 (Internet Protocol version 6, RFC 8200)

## TL;DR
Преемник IPv4. **128-битные адреса** (~3.4×10³⁸ — астрономически достаточно). **Упрощённый заголовок** (40 байт фиксированно, без options и checksum). **Нет фрагментации в маршрутизаторах** — только источник через PMTUD. **SLAAC** — автоконфигурация без DHCP. **Расширения** через extension headers. Развёртывание идёт с 1998, в 2026 ~40-50% мирового трафика.

## Какую проблему решает
IPv4 кончился: NAT — костыль, end-to-end сломан, port-forwarding для каждого сервиса. IPv6 даёт **достаточно адресов** для каждого устройства глобально, **прямую связность** без NAT, **упрощённую обработку** в маршрутизаторах.

## Как работает

**Адрес IPv6** — 128 бит, 8 групп по 16 бит в hex через `:`:
```
2001:0db8:85a3:0000:0000:8a2e:0370:7334
```
Сокращения:
- Лидирующие нули в группе можно опустить: `2001:db8:85a3:0:0:8a2e:370:7334`.
- Один блок подряд из нулей заменить на `::` (только один раз): `2001:db8:85a3::8a2e:370:7334`.
- `::1` — loopback (= IPv4 127.0.0.1).
- `::` — unspecified (= 0.0.0.0).

**Адресные блоки:**
- `2000::/3` — Global Unicast (вся «обычная» интернет-маршрутизация).
- `fe80::/10` — Link-Local (всегда назначается; для соседей по линку).
- `fc00::/7` (`fd00::/8` практически) — ULA (Unique Local Address, аналог 192.168.x).
- `ff00::/8` — Multicast (нет broadcast! заменён multicast'ом).

**Стандартный prefix length для подсети:** `/64`. Хост — `interface ID 64 бит` (часто из MAC через EUI-64).

**Заголовок IPv6** (40 байт):
```
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|Version|Traffic Class|         Flow Label                       |
|        Payload Length        | Next Header  | Hop Limit         |
|                                                                 |
|                Source Address (128 бит)                         |
|                                                                 |
|                Destination Address (128 бит)                    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

**Изменения от IPv4:**
- Нет header checksum (защищён L2/L4).
- **Next Header** заменяет Protocol; может указывать на **extension headers** (по Tanenbaum, стр. PDF 526–527 — 6 типов: **Hop-by-hop options, Destination options, Routing, Fragment, Authentication (AH), Encapsulating Security Payload (ESP)**) — затем сам payload.
- **Flow Label** — для QoS-классификации потоков.
- **Hop Limit** = TTL.

**SLAAC** (Stateless Address Autoconfiguration):
- Хост слышит RA (Router Advertisement) с префиксом /64.
- Сам себе назначает адрес: `prefix:: + interface_ID`.
- Без DHCP. (DHCPv6 опционально для DNS-серверов и т.п.)

**ICMPv6** (RFC 4443) — намного шире, чем v4: включает **NDP** (Neighbor Discovery, заменяет ARP), **MLD** (Multicast Listener Discovery, заменяет IGMP).

## Пример
**Современный смартфон в публичном Wi-Fi с IPv6:**
- AP объявляет prefix `2001:db8:abcd:1234::/64` через RA.
- Телефон делает SLAAC → его IPv6 = `2001:db8:abcd:1234:abcd:efff:fe12:3456`.
- Параллельно сохраняет IPv4 (DHCPv4) для дуал-стека.
- Запросы предпочтительно идут через IPv6 (RFC 6724 Happy Eyeballs); IPv4 — fallback.

## Связи
- **Базируется на:** [[Сетевой уровень]], [[IPv4]] (преемственность концепций), [[ICMP]] (расширен в ICMPv6).
- **Используется в:** [[IP-адресация и CIDR]] (CIDR применяется и к v6); [[Multicast routing]] (важнее в v6 — нет broadcast).
- **Соседи по уровню:** [[IPv4]] — предшественник, ещё активен; **dual-stack** — типичный режим.
- **Противопоставляется:** [[NAT]] — в v6 не нужен по умолчанию.

## Подводные камни
- **Развёртывание медленное:** провайдеры, оборудование, приложения — везде надо обновлять. К 2026 ~50% трафика, но 100% — далеко.
- **Дуал-стек** = две таблицы маршрутов, две firewall-конфигурации. Сложнее.
- **Privacy-extensions** (RFC 4941): по умолчанию SLAAC даёт стабильный адрес из MAC → tracking. Современные ОС используют **temporary** privacy-адреса.
- IPv6 ≠ автоматически безопасный: **те же** уязвимости, что и в v4.

## Дальше читать
- [[IPv4]] — для контраста.
- [[ICMP]] — ICMPv6 шире.
- Tanenbaum, гл. 5, §5.7.3 (стр. PDF 519–530).
