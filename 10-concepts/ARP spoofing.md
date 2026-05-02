---
title: ARP spoofing
aliases: ["ARP poisoning", "Отравление ARP"]
type: concept
layer: security
chapter: 8
difficulty: basic
prerequisites: ["[[ARP]]"]
related: ["[[Виды атак]]"]
tags: [networking, ch08]
---
# ARP spoofing (poisoning)

## TL;DR
Атака MITM в LAN. Злоумышленник шлёт поддельные **ARP-ответы**, привязывая свой MAC к IP-адресу gateway/жертвы. Жертвы кэшируют → шлют трафик через атакующего. Просто реализуется (`arpspoof`, `ettercap`), катастрофична: атакующий читает/меняет весь LAN-трафик.

## Какую проблему решает
ARP — **plain-text без аутентификации**. Любой узел в LAN может ответить «у меня IP X», и его поверят. Это известная проблема с самого 1982 г. — спустя 40+ лет всё ещё актуальна.

## Как работает

**Сценарий:**
1. В LAN: жертва (192.168.1.5), gateway (192.168.1.1), атакующий (192.168.1.50).
2. Атакующий шлёт жертве ARP-ответ: «192.168.1.1 has MAC = my MAC».
3. Жертва обновляет ARP-кэш → весь трафик к gateway идёт через атакующего.
4. Атакующий параллельно шлёт gateway: «192.168.1.5 has MAC = my MAC» → симметрично.
5. Атакующий посередине — читает, меняет, перенаправляет.

```mermaid
flowchart LR
  Victim -->|"трафик к gateway"| Attacker
  Attacker -->|"переслал к gateway"| Gateway
  Gateway -->|"ответ"| Attacker
  Attacker -->|"переслал жертве"| Victim
```

**Что атакующий может:**
- **Eavesdrop** на HTTP, неглазированном email, DNS.
- **Modify** content (inject JS в HTTP-страницы).
- **SSL strip** — заменить HTTPS на HTTP в редиректах (если нет HSTS).
- **DoS** — просто дропать трафик жертвы.

## Пример
**Кофейня с открытым Wi-Fi:**
- Атакующий с ноутом → `arpspoof -i wlan0 -t victim_ip gateway_ip`.
- Жертва не замечает — пакеты идут через атакующего.
- HTTPS-сайты защищены (TLS-cert не подделать без known CA).
- Но HTTP, неглазированные мессенджеры, app's без cert-pinning — уязвимы.
- Если жертва нажмёт «accept fake cert» — даже HTTPS взломан.

## Связи
- **Базируется на:** [[ARP]] (отсутствие аутентификации).
- **Используется в:** [[Виды атак]] (как пример MITM в LAN).
- **Соседи по уровню:** [[DNS spoofing]] — родственная атака на L7.
- **Противопоставляется:** **Dynamic ARP Inspection** на switch'е, **static ARP** для критичных серверов, **802.1X** auth.

## Подводные камни
- **Современный TLS + HSTS** делает ARP-spoofing менее опасным для веба, но **не устраняет** его. Все еще читается DNS, можно делать SSL strip на сайтах без HSTS.
- **Защита** на switch'е: DAI (Dynamic ARP Inspection) + DHCP snooping. Switch отслеживает legitimate IP-MAC bindings и дропает чужие ARP-ответы.
- **IPv6** — там ARP заменён NDP, но спроофинг тоже возможен (NDP spoofing), защита через **RA Guard, NDPMon**.

## Дальше читать
- [[ARP]] — что атакуется.
- [[DNS spoofing]] — соседняя атака.
- Tanenbaum, гл. 8, §8.2.3 (стр. PDF 827–840).
