---
title: NDP
aliases: ["Neighbor Discovery Protocol", "ICMPv6 NDP"]
type: protocol
layer: network
chapter: 5
difficulty: basic
prerequisites: ["[[IPv6]]", "[[ICMP]]"]
related: ["[[ARP]]", "[[DHCP]]"]
tags: [networking, ch05]
---
# NDP — Neighbor Discovery Protocol (RFC 4861)

## TL;DR
Замена ARP в IPv6, реализованная поверх **ICMPv6**. Делает: разрешение IPv6→MAC, обнаружение маршрутизаторов, **SLAAC** (автоконфигурация адреса), redirect. Также **DAD** (Duplicate Address Detection) — проверка уникальности своего адреса в LAN. Базис «без DHCP» в IPv6.

## Какую проблему решает
В IPv6:
- ARP не подходит (broadcast'а нет).
- DHCP опционален (через SLAAC можно без него).
- Нужны механизмы для discovery маршрутизатора, обнаружения дубликатов, перенаправления.

NDP объединяет всё это в один протокол поверх ICMPv6 multicast'ов.

## Как работает

**5 типов сообщений ICMPv6:**

| Тип | Имя | Назначение |
|---|---|---|
| 133 | **RS** Router Solicitation | хост: «есть кто-нибудь маршрутизатор?» |
| 134 | **RA** Router Advertisement | роутер: «я тут, prefix=X, флаги конфигурации» |
| 135 | **NS** Neighbor Solicitation | «у кого IPv6=X? сообщи MAC» (как ARP-request) |
| 136 | **NA** Neighbor Advertisement | «X = моё, MAC=Y» (как ARP-reply) |
| 137 | **Redirect** | «иди через другой gateway, не через меня» |

**Многоадресность вместо broadcast:** NS направляется на **solicited-node multicast address** (`ff02::1:ffXX:XXXX` — последние 24 бита target IP). Только хосты с подходящим адресом «слышат» — нагрузка меньше, чем broadcast ARP.

**SLAAC** (Stateless Address Autoconfiguration, RFC 4862):
1. Хост подключается к LAN, шлёт RS.
2. Роутер шлёт RA с prefix `2001:db8::/64` и флагами:
   - `M` (managed) — использовать DHCPv6 для адреса.
   - `O` (other config) — DHCPv6 для опций (DNS).
   - `A` (autonomous) — можно SLAAC.
3. Хост: `prefix:: + interface_ID` → свой IPv6.
4. **DAD:** хост шлёт NS на свой только что сгенерированный адрес. Если кто-то ответит NA → дубль, не использовать. Тишина → адрес уникален.

**Privacy extensions** (RFC 4941): SLAAC из MAC даёт стабильный адрес, который трекается. Modern OS генерируют temporary случайные **interface IDs**, обновляющиеся ежедневно.

## Пример
**Включился новый MacBook в офисе:**
1. Получает link-local `fe80::...` (always).
2. Шлёт **RS** на `ff02::2` (all routers multicast).
3. Получает **RA** от роутера: prefix `2001:db8:abcd:1234::/64`, A=1.
4. Генерирует interface ID (случайный для приватности): `2001:db8:abcd:1234:abcd:efff:fe12:3456`.
5. Шлёт **NS** на solicited-node multicast этого адреса (DAD).
6. Тишина → адрес уникален → начинает использовать.
7. Параллельно DHCPv6 (если флаг M=1) — для DNS-конфигурации.

## Связи
- **Базируется на:** [[IPv6]] (его ARP-замена), [[ICMP]] (поверх ICMPv6).
- **Используется в:** все IPv6-сети для адресной автоконфигурации.
- **Соседи по уровню:** [[ARP]] — IPv4-аналог; [[DHCP]] (DHCPv6) — может работать вместе для DNS-конфигурации.
- **Противопоставляется:** ARP в IPv4 — broadcast-based; NDP — multicast-based, более скромная нагрузка.

## Подводные камни
- **NDP-spoofing** (аналог ARP-spoofing) возможен: злоумышленник может прислать NA с фальшивым MAC или RA с подложным prefix → MITM. Защита — **RA Guard**, **DHCPv6 Guard**, **SeND** (Secure ND, RFC 3971 — мало используется).
- **RA с подложного источника** в публичных Wi-Fi — серьёзная угроза. Современные switch'и должны фильтровать.
- В мобильных и виртуализированных средах SLAAC может медлить из-за DAD (~1 с) — для real-time это слишком.

## Дальше читать
- [[ARP]] — IPv4-предшественник.
- [[IPv6]], [[ICMP]] — родительские протоколы.
- [[DHCP]] — может работать вместе.
- Tanenbaum, гл. 5, §5.7.4 (стр. PDF 530–531).
