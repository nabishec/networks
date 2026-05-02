---
title: Multicast Internet
aliases: ["IGMP", "PIM", "PIM-SM", "PIM-DM"]
type: protocol
layer: network
chapter: 5
difficulty: intermediate
prerequisites: ["[[Multicast routing]]"]
related: ["[[Anycast]]"]
tags: [networking, ch05]
---
# Multicast Internet (IGMP + PIM)

## TL;DR
Конкретная реализация [[Multicast routing|multicast]] в IP-интернете: **IGMP** (host↔router в LAN: подписки на группы) + **PIM** (router↔router: построение деревьев между маршрутизаторами). Работает на изолированных IP-областях (один оператор, корп. сеть, IPTV); в публичном интернете multicast практически не маршрутизируется.

## Какую проблему решает
Раздать одно потоковое видео тысячам зрителей **эффективно** — без отправки тысячи копий. IP-multicast: один пакет идёт по дереву к подписчикам.

## Как работает

### IGMP (Internet Group Management Protocol, RFC 3376)
**Host ↔ local router.** Хост сообщает локальному маршрутизатору, на какие multicast-группы он подписан.

- **Membership Report:** хост шлёт «я слушаю 239.1.1.10».
- **Membership Query:** маршрутизатор периодически опрашивает «есть кто-нибудь?».
- Если на сегменте нет ни одного слушателя — маршрутизатор перестаёт пересылать.

**IGMP snooping** на switch'е — оптимизация: switch смотрит на IGMP-сообщения и не флудит multicast в порты без слушателей.

### PIM (Protocol Independent Multicast, RFC 7761 для PIM-SM)
**Router ↔ router.** Строит дерево распределения.

- **PIM-DM (Dense Mode):** flood-and-prune. Сначала затопить всех; маршрутизаторы без подписчиков шлют **prune** обратно. Хорошо при плотных группах.
- **PIM-SM (Sparse Mode):** через **Rendezvous Point** (RP). Подписчики через свои маршрутизаторы регистрируются у RP. Источник тоже регистрируется через RP. После установления потока — переключение на shortest-path tree.
- **PIM-SSM (Source-Specific Multicast):** клиент подписывается на пару `(source, group)` — тогда дерево от каждого source отдельно, RP не нужен.

**RPF** (Reverse Path Forwarding) — маршрутизатор пересылает multicast только если он пришёл с интерфейса, ведущего обратно к источнику.

## Пример
**Корпоративный IPTV в большой компании:**
- Сервер вещает на 239.1.1.10.
- Сотрудник включил плеер → IGMP report → локальный маршрутизатор.
- Локальный → PIM-SM регистрация у RP.
- Поток идёт через RP → потом переключается на shortest path.
- Видео доставлено всем, кто подписан, по дереву, без дубликатов на промежуточных линках.

**Финансовые market data feeds:** trading-firms используют multicast для миллисекундной доставки одного фида тысячам подписчиков в DC. Public internet здесь не нужен.

## Связи
- **Базируется на:** [[Multicast routing]] (общая идея), [[OSPF]] (для unicast routing, на котором PIM строит RPF).
- **Используется в:** IPTV-провайдеры, корпоративные стримы, finance, IoT-публикация.
- **Соседи по уровню:** **MLD** (Multicast Listener Discovery) в IPv6 — аналог IGMP через ICMPv6.
- **Противопоставляется:** unicast (одна копия на каждого) и pub-sub поверх unicast (Kafka и т.п.) — последнее в DC заменило multicast.

## Подводные камни
- **Публичный интернет не маршрутизирует multicast** — операторы боятся overhead и DDoS-векторов. ASM (Any-Source Multicast) между AS почти отсутствует.
- **MSDP** (Multicast Source Discovery Protocol) пытался решить inter-domain multicast — едва используется.
- В IPv6 multicast встроен глубже (нет broadcast, MLD), но та же проблема публичной маршрутизации.
- **SSM** проще для inter-domain (нет shared RP), но требует поддержки приложением (IGMPv3+).

## Дальше читать
- [[Multicast routing]] — общая теория.
- Tanenbaum, гл. 5, §5.7.8 (стр. PDF 551–552).
