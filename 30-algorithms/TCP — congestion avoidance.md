---
title: TCP — congestion avoidance
aliases: ["Congestion avoidance", "Fast retransmit", "Fast recovery", "TCP Reno", "TCP New Reno"]
type: algorithm
layer: transport
chapter: 6
difficulty: intermediate
prerequisites: ["[[TCP — slow start]]", "[[AIMD]]"]
related: ["[[CUBIC TCP]]", "[[BBR]]"]
tags: [networking, ch06]
---
# TCP — congestion avoidance + Fast Retransmit/Recovery

## TL;DR
После [[TCP — slow start|slow start]] TCP переходит в **линейный рост** cwnd (`+1 MSS` за RTT) — это AIMD-AI. При потере: **timeout** → старт slow start с самого начала; **3 dup-ACK** → **fast retransmit** (немедленный повтор) + **fast recovery** (cwnd /= 2, продолжить с CA). Базис TCP Reno (1990) и New Reno (1996).

## Какую проблему решает
Slow start — для разгона. Когда подходим к ёмкости сети (ssthresh), агрессивный рост вреден: следующее удвоение → потери. Нужна **более скромная** стратегия после ssthresh — отсюда linear growth (AI). Также при изолированных потерях не хочется сбрасывать всё с нуля — отсюда fast retransmit/recovery.

## Как работает

### Congestion Avoidance (AI):
- cwnd ≥ ssthresh.
- Каждый ACK: cwnd += MSS²/cwnd ≈ +1 MSS за **RTT** (не за каждый ACK).

### Fast Retransmit:
- Получатель: пришёл out-of-order сегмент → шлёт duplicate ACK (тот же ack-num).
- Отправитель видит **3 dup ACKs** на один и тот же seq → высокая вероятность потери, не reorder.
- **Не ждёт RTO**, переслать немедленно потерянный сегмент.

### Fast Recovery (RFC 2581 — Reno):
- При fast retransmit: ssthresh = cwnd/2; cwnd = ssthresh (не 1 MSS).
- Каждый dup-ACK после: cwnd += 1 MSS (inflate window — нагрузка от дубликатов компенсируется).
- При first non-dup ACK (recover): cwnd = ssthresh.
- Возвращение в congestion avoidance.

```
cwnd
                potеря (3 dup-ACK)
       ssthresh ╲↓
  ╱╲    ╱╲          ↘ fast recovery
 ╱  ╲  ╱  ╲   AI     ╱
╱    ╲╱    ╲────╲  ╱  ← linear growth (AI)
              ╲ ╱
               × loss
```

### Различие между Reno и New Reno
- **Reno:** при множественных потерях в окне fast recovery срабатывает только на одну.
- **New Reno (RFC 6582):** partial ACK во время recovery → переслать следующий потерянный без выхода из recovery.

### TCP Tahoe (1988)
- Старая версия: при любой потере → slow start с cwnd=1.
- Заменена Reno благодаря fast recovery.

## Пример
**Стабильный download через TCP Reno на «50 мс RTT, 100 Мбит/с»:**
- Slow start: за ~7 RTT cwnd достиг ~ssthresh.
- CA: linear growth.
- Случайная потеря (1 сегмент) → 3 dup-ACK → fast retransmit → cwnd /= 2 → CA продолжается.
- График cwnd — характерный «зуб пилы» AIMD.

При **timeout** (полная потеря): cwnd → 1, slow start заново. Дороже.

## Связи
- **Базируется на:** [[TCP — slow start]] (предыдущая фаза), [[AIMD]] (это AI), [[ARQ]] (fast retransmit — оптимизация ARQ).
- **Используется в:** TCP Reno, New Reno, CUBIC (модифицированная AI).
- **Соседи по уровню:** [[CUBIC TCP]] — современный default Linux; [[BBR]] — model-based, без AIMD.
- **Противопоставляется:** TCP Tahoe — не имел fast recovery.

## Подводные камни
- На high-BDP сетях AIMD-восстановление **очень медленное**: cwnd /= 2 → линейный рост → много RTT до полной утилизации. Поэтому **CUBIC** (кубический рост) лучше.
- **3 dup-ACK** не всегда означают потерю — может быть reorder. Современные реализации используют **early retransmit, RACK** для лучшей детекции.
- **SACK** (RFC 2018) делает fast recovery эффективнее — отправитель знает, что потеряно конкретно.

## Дальше читать
- [[TCP — slow start]] — что было до.
- [[CUBIC TCP]], [[BBR]] — следующие поколения.
- Tanenbaum, гл. 6, §6.5.10 (стр. PDF 643–654).
