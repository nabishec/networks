---
title: ARP
aliases: ["Address Resolution Protocol"]
type: protocol
layer: data-link
chapter: 5
difficulty: basic
prerequisites: ["[[IP-адресация и CIDR]]", "[[MAC-адрес]]", "[[Ethernet — IEEE 802.3]]"]
related: ["[[DHCP]]", "[[IPv6]]"]
tags: [networking, ch05]
---
# ARP — Address Resolution Protocol (RFC 826)

## TL;DR
Связывает **IPv4-адрес → MAC-адрес** в локальной сети. Когда хосту нужно отправить IP-пакет внутри LAN, он шлёт **broadcast ARP-запрос** «у кого IP X, ответьте мне». Хост с этим IP отвечает unicast'ом со своим MAC. Кешируется в **ARP-таблице** с timeout (~5 мин). В IPv6 ARP заменён на **NDP** (Neighbor Discovery Protocol) поверх ICMPv6.

## Какую проблему решает
IP-адрес — для глобальной идентификации хоста; MAC — для физической доставки в LAN. Хост знает IP получателя, но Ethernet-фрейм требует **MAC**. ARP — мост: преобразует одно в другое.

## Как работает

**ARP request:**
```
Source MAC: aa:bb:cc:dd:ee:ff (отправитель)
Destination MAC: ff:ff:ff:ff:ff:ff (broadcast)
ARP: who has 192.168.1.10? Tell 192.168.1.5
```

**ARP reply** (unicast):
```
Source MAC: 11:22:33:44:55:66 (хост 192.168.1.10)
Destination MAC: aa:bb:cc:dd:ee:ff (исходному отправителю)
ARP: 192.168.1.10 is at 11:22:33:44:55:66
```

**ARP-таблица:** `IP ↔ MAC ↔ TTL`. Истекает через ~5 мин. Можно посмотреть `arp -a` или `ip neigh`.

**Gratuitous ARP** — хост анонсирует своё IP→MAC без запроса. Используется при изменении IP, для обновления соседей; также при VRRP/HSRP failover.

## Пример
**Открыть google.com:**
1. DNS → 142.250.180.78.
2. Хост: 142.250.180.78 — это **не** в моей подсети `192.168.1.0/24` → отправлять через **default gateway** `192.168.1.1`.
3. Нужно найти MAC `192.168.1.1`. Смотрим ARP-кеш. Если нет — ARP request.
4. Роутер отвечает.
5. Хост: формирует Ethernet-фрейм `dst MAC = MAC роутера`, `IP dst = 142.250.180.78`. Отправляет.
6. Роутер пересылает дальше.

**Заметь:** ARP запрашивается **только** для соседа в LAN (gateway, не далёкий хост). MAC роутера, не MAC google.com.

## Связи
- **Базируется на:** [[MAC-адрес]] (что ищем), [[IP-адресация и CIDR]] (что у нас есть), [[Ethernet — IEEE 802.3]] (broadcast-фреймы).
- **Используется в:** все IPv4 LAN; **gratuitous ARP** в VRRP/HSRP, при failover.
- **Соседи по уровню:** **NDP** в IPv6 — функционально аналогично, реализовано через ICMPv6.
- **Противопоставляется:** в IPv6 ARP'а нет — заменён NDP.

## Подводные камни
- **ARP spoofing / poisoning** — атака MITM в LAN: злоумышленник отвечает на ARP-запросы фальшивыми MAC, заворачивая трафик через себя. Защита — **Dynamic ARP Inspection** на switch'е, **static ARP** для критичных серверов.
- ARP — **plain-text без аутентификации**. Любая host в LAN может говорить «у меня IP X», и его поверят.
- ARP-кеш истекает → периодически новые запросы → небольшой broadcast overhead. На очень больших L2-сегментах это может быть заметно.

## Дальше читать
- [[MAC-адрес]] — что мы получаем.
- [[IP-адресация и CIDR]] — IP-адреса.
- [[IPv6]] — преемник без ARP.
- Tanenbaum, гл. 5, §5.7.4 (стр. PDF 530–531).
