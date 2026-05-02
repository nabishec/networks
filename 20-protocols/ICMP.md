---
title: ICMP
aliases: ["Internet Control Message Protocol", "ICMPv4", "ICMPv6"]
type: protocol
layer: network
chapter: 5
difficulty: basic
prerequisites: ["[[IPv4]]", "[[IPv6]]"]
related: ["[[Internetworking — фрагментация]]"]
tags: [networking, ch05]
---
# ICMP — Internet Control Message Protocol

## TL;DR
Служебный протокол сетевого уровня для **сообщений о состоянии** и **диагностики**. Сам ICMP-пакет идёт в IP-payload (Protocol=1 для ICMPv4, 58 для ICMPv6). Главные сообщения: **Echo Request/Reply** (ping), **Destination Unreachable**, **Time Exceeded** (TTL = 0, основа traceroute), **Fragmentation Needed** (для PMTUD), **Redirect** (маршрутизатор подсказывает лучший путь).

## Какую проблему решает
IP сам по себе **не сообщает об ошибках** доставки. Если пакет потерялся — отправитель не знает. ICMP — стандартный канал, через который маршрутизаторы и хосты сообщают друг другу: «не дошло, потому что …». Без него debugging сети был бы невозможен.

## Как работает

**ICMPv4 — главные типы:**

| Тип | Имя | Когда |
|---|---|---|
| 0 | Echo Reply | ответ на ping |
| 3 | Destination Unreachable | host/network/port unreachable |
| 5 | Redirect | «иди через другой gateway» |
| 8 | Echo Request | ping |
| 11 | Time Exceeded | TTL=0 (для traceroute) |
| 12 | Parameter Problem | malformed заголовок |

**Destination Unreachable codes:**
- 0 — net unreachable
- 1 — host unreachable
- 3 — port unreachable (для UDP)
- 4 — **fragmentation needed** (для PMTUD)
- 9, 10, 13 — admin filter

**Структура ICMP-сообщения:**
```
+--------+--------+------------------+
|  Type  |  Code  |    Checksum      |
+--------+--------+------------------+
|         Message body              |
+-----------------------------------+
```

Тело часто содержит **первые 8 байт исходного пакета**, чтобы отправитель понял, к какой сессии относится ошибка.

**ICMPv6** (RFC 4443):
- Заменяет ARP (через NDP — Neighbor Discovery Protocol).
- MLD для multicast-membership.
- RA/RS для SLAAC.

## Пример
**ping 8.8.8.8:**
- Хост шлёт ICMP Echo Request (type 8).
- 8.8.8.8 отвечает Echo Reply (type 0).
- Видим RTT в выводе.

**traceroute 8.8.8.8:**
- Хост шлёт UDP-пакеты (или ICMP) с TTL=1, 2, 3, …
- Каждый маршрутизатор на пути уменьшает TTL → если стало 0, шлёт ICMP Time Exceeded к источнику.
- Источник по этим ICMP видит каждый hop по очереди.

**PMTUD:** ICMP «Fragmentation needed, MTU=X» — сигнализирует уменьшить MSS.

## Связи
- **Базируется на:** [[IPv4]], [[IPv6]] (несётся в IP-payload).
- **Используется в:** [[Internetworking — фрагментация]] (PMTUD), ping, traceroute, диагностика.
- **Соседи по уровню:** **NDP** в IPv6 (через ICMPv6), **IGMP** для multicast (отдельный протокол в v4, в v6 — MLD через ICMPv6).
- **Противопоставляется:** TCP/UDP — payload-протоколы; ICMP — control-плоскость IP.

## Подводные камни
- **Многие провайдеры/firewall'ы блокируют ICMP** (по ошибочной идее «безопасности»). Это ломает PMTUD, traceroute, ping. Часто — необъяснимые «зависания» сетей.
- **ICMP echo не гарантия рабочего сервиса:** хост отвечает на ping, но веб-сервер может быть упавшим.
- **ICMP redirect** — потенциальный вектор атаки (фишинг маршрута). На современных сетях обычно отключён.
- **Smurf-атака** (1990-е): отправка ICMP Echo на broadcast с подложенным src → DDoS. Современные сети броадкастов в WAN не пропускают.

## Дальше читать
- [[Internetworking — фрагментация]] — где ICMP критично.
- [[IPv4]], [[IPv6]] — родительские протоколы.
- Tanenbaum, гл. 5, §5.7.4 (стр. PDF 530–531).
