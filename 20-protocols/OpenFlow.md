---
title: OpenFlow
aliases: ["OpenFlow protocol"]
type: protocol
layer: network
chapter: 5
difficulty: intermediate
prerequisites: ["[[SDN — программно-конфигурируемые сети]]"]
related: ["[[Маршрутизатор]]"]
tags: [networking, ch05]
---
# OpenFlow

## TL;DR
Протокол связи между **SDN-контроллером** и **switch'ом** (southbound API). Контроллер ставит **flow-правила** в таблицы switch'а: `match (по полям пакета) → actions (forward, drop, modify, encap)`. Switch применяет первое совпадающее правило. Стандартизован Open Networking Foundation (ONF), 1.0 в 2009. Породил движение SDN, сейчас постепенно сменяется P4/gNMI, но остаётся учебным эталоном.

## Какую проблему решает
SDN-контроллеру нужен стандартный способ программировать switch'и от любого вендора. До OpenFlow каждый вендор имел свои CLI и protocols. OpenFlow дал **общий язык**: «открой таблицу, поставь правило, удали правило».

## Как работает

**Flow-правило (упрощённо):**
- **Match fields:** in_port, src/dst MAC, src/dst IP, src/dst port, EtherType, VLAN ID, TCP flags, и т.д. — поддерживает wildcard.
- **Priority:** при многих match'ах берётся высший priority.
- **Counters:** подсчёт пакетов/байтов.
- **Actions:** forward to port N, drop, modify (set field), encapsulate, group.
- **Timeout:** soft (idle), hard (absolute).

**Жизненный цикл пакета на switch'е:**
1. Пакет приходит в порт.
2. Ищется match в таблице.
3. Найдено → выполнить actions.
4. Не найдено → отправить контроллеру через **PACKET-IN** (или drop по дефолту).
5. Контроллер проанализировал, поставил правило, ответил **FLOW-MOD**.
6. Следующий такой пакет уже обрабатывается без участия контроллера.

**Версии:**
- **OpenFlow 1.0** (2009) — одна таблица, базовые match.
- **OpenFlow 1.3** (2012) — multiple flow tables (pipeline), group tables, IPv6, MPLS.
- **OpenFlow 1.5** (2014) — egress-таблицы, расширения.
- Развитие далее ушло в **P4** (более гибкая программируемость pipeline'а).

## Пример
**SDN-контроллер настраивает балансировщик:**
- Видит пакет TCP к VIP 10.0.0.1:80 на switch S1.
- Решает: «отправить на сервер 10.1.1.5».
- Шлёт FLOW-MOD: `match dst=10.0.0.1, dst_port=80 → set dst=10.1.1.5, fwd port 3, priority 100, idle_timeout 60s`.
- Все следующие SYN на этот VIP идут к серверу 5 — без участия контроллера.

При следующих сессиях балансировщик может выбрать другой сервер → новый FLOW-MOD.

## Связи
- **Базируется на:** [[SDN — программно-конфигурируемые сети]] (это его southbound-протокол).
- **Используется в:** ONOS, OpenDaylight, Ryu, Floodlight контроллеры; Open vSwitch (программный switch).
- **Соседи по уровню:** **NETCONF/YANG**, **gNMI**, **P4Runtime** — более новые альтернативы southbound.
- **Противопоставляется:** vendor-specific CLI/SNMP — закрытые, не SDN.

## Подводные камни
- **Производительность:** flow-таблицы реализуются в TCAM, который дорог и ограничен в объёме (десятки тысяч записей). На крупных сетях — ограничение.
- **Контроллер-bottleneck:** если каждый новый flow требует обращения к контроллеру (PACKET-IN/FLOW-MOD), latency и нагрузка значительные. Решение — **proactive** правила (заранее ставить общие).
- **Не подходит** для сложного pipeline'а (encap+decap+complex header rewrite); для этого появился P4.

## Дальше читать
- [[SDN — программно-конфигурируемые сети]] — общий контекст.
- Tanenbaum, гл. 5, §5.6 (стр. PDF 495–498).
