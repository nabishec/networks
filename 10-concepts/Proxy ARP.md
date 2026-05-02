---
title: Proxy ARP
aliases: ["Proxy ARP", "RARP", "BOOTP"]
type: concept
layer: network
chapter: 5
difficulty: basic
prerequisites: ["[[ARP]]"]
related: ["[[DHCP]]"]
tags: [networking, ch05]
---
# Proxy ARP, RARP/BOOTP — историческое

## TL;DR
**Proxy ARP** — приём, при котором маршрутизатор/host отвечает на ARP-запросы от **имени** другого хоста (того, кого нет в данной L2-сети). Полезен для legacy-устройств, mobile IP, firewall-NATраздачи. **RARP** (Reverse ARP, RFC 903) и **BOOTP** (RFC 951) — предшественники DHCP для автоконфигурации хоста.

## Какую проблему решает

### Proxy ARP
В нормальном scenario ARP-запрос для удалённого IP не получает ответа (хост не в LAN). Proxy ARP: **router** видит запрос про IP в **другой** подсети, **отвечает** своим MAC. Запрашивающий шлёт фрейм роутеру → роутер форвардит как обычно через L3.

Применения:
- **Mobile IP** (host moves to другой network — proxy ARP делает его «локально доступным»).
- **Firewall**, разделяющий одну подсеть на две — proxy ARP мостит ARP между ними.
- **PPP с unnumbered links** — нет own subnet, proxy ARP делает чужой адрес доступным локально.

### RARP (Reverse ARP)
- Хост знает свой MAC, не знает IP.
- Шлёт RARP-broadcast с MAC; RARP-server отвечает с IP.
- **Заменён DHCP** (1993+) — предоставляет больше parameters (gateway, DNS, ...).

### BOOTP (Bootstrap Protocol)
- Между RARP и DHCP по сложности.
- Дополнительно: server, kernel filename для PXE-boot diskless workstation.
- Поверх UDP, с replication; имел концепцию **lease** хоть и static.
- DHCP — **superset** BOOTP, обратно совместим.

## Как работает (Proxy ARP)

**Сценарий:**
- Host A в subnet 192.168.1.0/24, MAC A.
- Host B в subnet 192.168.2.0/24, MAC B.
- Router R с интерфейсами в обе подсети.

**A хочет связаться с B (192.168.2.10):**
- A не знает, что B в другой subnet (например, mis-configured netmask).
- A шлёт ARP-broadcast: «who has 192.168.2.10».
- B не слышит (разные L2-сегменты).
- **R слышит** → знает «192.168.2.10 у меня через интерфейс X» → **отвечает A своим MAC**.
- A поверил → шлёт фрейм R'у.
- R форвардит как обычно.

A не знает, что говорит с router'ом, не B; всё работает.

## Пример
**Mobile IP** (RFC 5944):
- Mobile node переехал из home network в foreign.
- В home network — **home agent** (роутер) делает proxy ARP для mobile node's IP → пакеты приходят к home agent.
- Home agent **тунелирует** пакеты в foreign network к mobile node.
- Mobile видит свой home IP в любом месте.

## Связи
- **Базируется на:** [[ARP]] (механизм), [[DHCP]] (заменил RARP/BOOTP).
- **Используется в:** Mobile IP, некоторые firewall'ы, legacy compatibility.
- **Соседи по уровню:** **Gratuitous ARP** — host анонсирует свой IP-MAC (для VRRP, FailOver).
- **Противопоставляется:** native subnetting — проще, без proxy.

## Подводные камни
- **Proxy ARP опасен** при misconfig — ARP-loops, конфликт IP.
- **Modern best practice:** избегать proxy ARP, использовать proper subnetting.
- В **современных DC** — VXLAN/EVPN решают подобные проблемы elegantly без proxy ARP.
- **DHCP** **полностью заменил** RARP/BOOTP — но в PXE-boot по-прежнему может быть BOOTP-like flow.

## Дальше читать
- [[ARP]] — основа.
- [[DHCP]] — преемник RARP/BOOTP.
- Tanenbaum, гл. 5, §5.7.4 (стр. PDF 533–535).
