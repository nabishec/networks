---
title: Fair queueing (FQ, WFQ, DRR)
aliases: ["Fair queueing", "FQ", "WFQ", "Weighted fair queueing", "DRR", "Deficit Round Robin"]
type: algorithm
layer: network
chapter: 5
difficulty: intermediate
prerequisites: ["[[Перегрузка сети]]"]
related: ["[[DiffServ]]", "[[Token bucket]]", "[[RED и AQM]]"]
tags: [networking, ch05]
---
# Fair queueing — FQ, WFQ, DRR

## TL;DR
Семейство планировщиков пакетов на маршрутизаторе, разделяющих ёмкость линка **между потоками** так, чтобы один «жадный» поток не утопил остальных. **FIFO** — все в одной очереди, нечестно. **Fair queueing** — отдельная очередь per-flow с round-robin. **WFQ** — то же с весами. **DRR** (Deficit Round Robin) — практичная реализация на железе. Базис QoS-планирования.

## Какую проблему решает
**FIFO** + drop tail: один TCP-поток с большим cwnd может занять весь буфер → другие потоки видят drops, замедляются, страдают. Это **несправедливо**: правила игры не зависят от количества потоков.

Fair queueing разделяет ресурс **по потокам**, не «по байтам».

## Как работает

### Bit-by-bit round robin (теоретический)
- Представим: «крутится колесо», на каждом обороте каждая активная очередь даёт 1 бит.
- Если в очереди A 100 байт ждут, в B 200 байт — за время обработки 1 байта выходит 1/2 байта от каждой.
- **Идеальный max-min fairness**.

Не реализуем (бит за раз — нет).

### Fair Queueing (FQ, Demers-Keshav-Shenker, 1989)
- Имитирует bit-by-bit RR через **finishing-time computation**: для каждого пакета считаем «когда он бы вышел при bit-by-bit».
- Передаётся пакет с минимальным finishing time.
- Per-flow queue (по 5-tuple для TCP/UDP).

### Weighted Fair Queueing (WFQ)
- То же, но потоки имеют **веса**. Поток с весом 2 получает в 2 раза больше ёмкости.
- Используется в DiffServ AF-классах.

### Deficit Round Robin (DRR, Shreedhar-Varghese, 1996)
- Для эффективной аппаратной реализации.
- Каждой очереди — **квант** (например, 1500 байт) и **deficit counter**.
- Round-robin: если deficit + quant ≥ length(head_packet) → передаём, deficit -= length; иначе deficit += quant, идём дальше.
- O(1) на пакет.

```
очередь A: ▓ ▓ ▓ ▓ (вес 2)
очередь B: ▓ (вес 1)
WFQ выход: A A B A A B A A B
```

## Пример
**Linux** в современных дистрибутивах: `fq_codel` или `cake` qdisc — **Fair Queueing + AQM** в одном:
- Per-flow queue (FQ) → справедливо.
- CoDel внутри каждой → борьба с bufferbloat.
- Включается обычно автоматически на исходящих интерфейсах.

**В DiffServ:**
- AF-классы (AF1x..AF4x) обслуживаются WFQ с разными весами.
- EF — отдельная priority queue (вне WFQ).

## Связи
- **Базируется на:** [[Перегрузка сети]] (инструмент справедливости при нагрузке).
- **Используется в:** [[DiffServ]] (AF-классы через WFQ), Linux fq_codel/cake, Cisco WRED+WFQ, CDN edges.
- **Соседи по уровню:** [[Token bucket]] (rate limiting per-flow), [[RED и AQM]] (управление длиной очередей).
- **Противопоставляется:** **FIFO + drop tail** — простой, но несправедливый и провоцирует bufferbloat.

## Подводные камни
- **«Поток»** — определяется по 5-tuple. Атакующий с многими src-port'ами может получить непропорциональную долю → fq_codel ограничивает per-flow buffer'ы.
- **Memory cost** — каждый flow требует свою queue. На high-end DC switch'ах это десятки тысяч очередей.
- **WFQ ≠ priority** — WFQ даёт **долю**, не приоритет. Priority queue (EF в DiffServ) — отдельный механизм поверх.
- **Strict priority** + WFQ комбинируется: EF идёт в strict priority queue, остальное — в WFQ.

## Дальше читать
- [[Перегрузка сети]] — общий контекст.
- [[DiffServ]] — главный потребитель WFQ.
- Tanenbaum, гл. 5, §5.4.3 (стр. PDF 466–473).
