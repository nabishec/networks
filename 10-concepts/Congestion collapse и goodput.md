---
title: Congestion collapse и goodput
aliases: ["Congestion collapse", "Goodput"]
type: concept
layer: network
chapter: 5
difficulty: basic
prerequisites: ["[[Перегрузка сети]]"]
related: ["[[AIMD]]", "[[Bufferbloat]]"]
tags: [networking, ch05]
---
# Congestion collapse и goodput

## TL;DR
**Goodput** — полезная пропускная способность для приложения (bytes useful data / time). **Throughput** — чисто пакеты в среду, **включая** retransmissions и прочий «balloон». В перегруженной сети throughput может быть высокий (всё что-то шлют), а goodput — близко к нулю (всё повторяется бесконечно). **Congestion collapse** — это коллапс **goodput**, исторически произошёл в интернете 1986 г. до изобретения TCP-congestion-control.

## Какую проблему решает
Различение **throughput** vs **goodput** — фундаментально для понимания, **зачем** TCP делает congestion control. Без CC сеть могла бы быть «занята», но не делать полезной работы.

## Как работает

**Definitions:**
- **Throughput** — bits/sec проходящих через канал. Считается на физ-уровне.
- **Goodput** — bits/sec **полезного** payload, доставленных адресату **успешно** (не дублей, не пакетов с ошибкой).

**Goodput < Throughput**, потому что:
- TCP-headers (~40 байт overhead).
- Retransmissions (потерянные пакеты).
- ACK'и в обратном направлении.

**Congestion collapse:**
- При перегрузке потери растут → больше retransmits → ещё больше offered load → больше потерь.
- В пределе: всё что в канале — retransmits. Goodput → 0, throughput на пределе.
- В буферах сидят дубликаты, успевшие устареть.

**Жизнь интернета 1986** (Tanenbaum, стр. 446):
- Backbone: 56 кбит/с между LBL и UC Berkeley.
- Throughput реально использован, но **goodput упал в 1000 раз** до 40 бит/с (!).
- Причина: TCP без congestion control + слабые маршрутизаторы.
- **Van Jacobson решил** через TCP slow start + congestion avoidance (RFC 1122, 1989).

```
goodput
  100% ─┐╲
        │ ╲___ ideal
        │     ╲___
        │         ╲
   0% ──┴──────────── offered load
          knee  cliff
                ↑
                congestion collapse
```

## Пример
**Современный сценарий:**
- Канал 100 Мбит/с, RTT 50 мс.
- Без congestion control: 100 потоков по 10 Мбит/с = 1 Гбит offered.
- Throughput линка = 100 Мбит/с (физ. предел).
- Goodput при 90% loss → 10 Мбит/с → каждый поток получает 100 кбит/с (в теории).
- В реальности — все retransmit'ят → goodput ещё ниже.
- TCP сам себя замедлит через AIMD → goodput выравнивается к ~95 Мбит/с (close to throughput).

## Связи
- **Базируется на:** [[Перегрузка сети]].
- **Используется в:** [[AIMD]] (предотвращение collapse), [[Bufferbloat]] (related modern problem).
- **Соседи по уровню:** **fairness** — другая метрика congestion-control (как делится).
- **Противопоставляется:** «throughput as performance» — обманчиво в перегрузке.

## Подводные камни
- **Goodput app-level vs network-level** различается. Например, HTTP/2 multiplexes 100 streams — если один блокируется HoL, его goodput=0, у других нормальный. Aggregate ≠ individual.
- **Modern Linux nedp.** показывает throughput, не goodput.
- В **DC-сетях с low-latency requirements** даже малая буферизация → goodput-deg для micro-bursts.

## Дальше читать
- [[Перегрузка сети]] — общая картина.
- [[AIMD]] — решение.
- [[Bufferbloat]] — современная вариация.
- Tanenbaum, гл. 5, §5.3.1 (стр. PDF 446).
