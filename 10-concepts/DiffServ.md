---
title: DiffServ
aliases: ["Differentiated Services", "DSCP", "Дифференцированное обслуживание"]
type: concept
layer: network
chapter: 5
difficulty: basic
prerequisites: ["[[QoS vs QoE]]"]
related: ["[[IntServ и RSVP]]", "[[Token bucket]]", "[[Сетевой нейтралитет]]"]
tags: [networking, ch05]
---
# DiffServ — Differentiated Services (RFC 2474, 2475)

## TL;DR
Альтернатива [[IntServ и RSVP|IntServ]] — **per-class** QoS вместо per-flow. Пакеты помечаются 6-битным **DSCP** (Differentiated Services Code Point) в IP-заголовке (часть TOS/Traffic Class). Маршрутизаторы применяют **per-hop behavior (PHB)** на основе DSCP — приоритет, очереди, drop-policy. **Не хранит state per-flow** — только конфигурация классов. Доминирующий QoS-механизм в реальных сетях.

## Какую проблему решает
IntServ не масштабируется (см. [[IntServ и RSVP]]). DiffServ переносит QoS в **простую табличку**: пакет имеет метку → маршрутизатор знает, как с ним обращаться. Не нужно резервировать что-либо заранее, не нужно signaling-протоколов в ядре.

## Как работает

**DSCP (6 бит):**
- 64 возможных значения.
- Стандартизованные классы:
  - **EF** (Expedited Forwarding, DSCP 46) — голос, низкая задержка/jitter.
  - **AF1x–AF4x** (Assured Forwarding) — 4 класса × 3 уровня drop precedence.
  - **CS0–CS7** (Class Selector) — обратная совместимость со старым IP Precedence.
  - **DSCP 0** (default) — best-effort.

**Per-Hop Behavior (PHB):**
- На каждом маршрутизаторе DSCP → правило: какая очередь, приоритет, drop-policy.
- **EF** → отдельная priority queue, минимум задержки.
- **AF** → weighted round-robin по классам, RED по drop precedence.
- **BE** → обычная очередь.

**Доменная модель:**
- **DiffServ-домен** = область одного оператора с согласованной политикой.
- **Edge-маршрутизатор** делает **classification** (определяет, в какой класс трафик) + **marking** (ставит DSCP) + **policing** (token bucket на пакеты класса).
- **Core-маршрутизаторы** только применяют PHB по DSCP.

```mermaid
flowchart LR
  Src[Источник] --> Edge[Edge: classify+mark+police]
  Edge --DSCP-->|46 (EF)| Core1[Core: priority queue]
  Edge --DSCP-->|0 (BE)| Core1
  Core1 --> Edge2[Edge выходит]
```

## Пример
**Корпоративный VoIP в DiffServ-домене:**
- IP-телефон шлёт RTP-пакеты с **DSCP 46 (EF)**.
- Edge-switch проверяет: трафик от телефона = EF → пропускает; от ноутбука = BE.
- Core-роутер: EF идёт в priority queue → задержка <5 мс; HTTP-трафик в обычную очередь — задержка 10–50 мс при загрузке.
- Голосу гарантирован приоритет.

**Wi-Fi WMM** — это DiffServ на L2: 4 класса доступа (BK/BE/VI/VO) с разными AIFS/CW.

## Связи
- **Базируется на:** [[QoS vs QoE]] (механизм её достижения), [[Token bucket]] (для policing).
- **Используется в:** все enterprise-сети с QoS, mobile carriers (LTE QCI), [[MPLS]] (DSCP→EXP-биты).
- **Соседи по уровню:** [[IntServ и RSVP]] — конкурент.
- **Противопоставляется:** IntServ per-flow; DiffServ per-class. Масштабируется лучше, но без жёстких гарантий.

## Подводные камни
- **DSCP не сохраняется** на границах AS публичного интернета — провайдеры **сбрасывают** или переопределяют. QoS работает **внутри** одной сети.
- Если все классы перегружены — приоритеты ничего не дают. DiffServ работает только когда **есть свободная ёмкость**, которую можно перераспределить.
- **Сетевой нейтралитет** ([[Сетевой нейтралитет]]) — DiffServ может использоваться нейтрально (приоритет голосу — не дискриминация контента) или нет (приоритет «своему» сервису).

## Дальше читать
- [[IntServ и RSVP]] — соперник.
- [[Token bucket]] — для policing.
- Tanenbaum, гл. 5, §5.4.5 (стр. PDF 477–480).
