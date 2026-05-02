---
title: RED и AQM
aliases: ["Random Early Detection", "AQM", "Active Queue Management", "CoDel", "fq_codel"]
type: algorithm
layer: network
chapter: 5
difficulty: intermediate
prerequisites: ["[[Перегрузка сети]]"]
related: ["[[ECN]]", "[[TCP]]"]
tags: [networking, ch05]
---
# RED и AQM

## TL;DR
**Active Queue Management** — превентивный drop пакетов в маршрутизаторе **до** переполнения буфера, чтобы дать TCP сигнал о congestion раньше. **RED** (Random Early Detection, Floyd & Jacobson 1993): начиная с порогового размера очереди, дроп с вероятностью, растущей линейно с заполнением. Лучше drop tail. Современные потомки — **CoDel** (Controlled Delay) и **fq_codel** в Linux.

## Какую проблему решает
**Drop tail** (стандартное FIFO до переполнения, потом drop) проблемен:
- Все потоки штрафуются одинаково — **синхронизация** TCP (все cwnd падают одновременно, потом восстанавливаются).
- TCP узнаёт о congestion **поздно** — буфер уже полон, пакеты летели через большую очередь.
- **Bufferbloat:** большие буферы → большие задержки до drop'а.

AQM решает: dropping **до** переполнения, **случайно**, чтобы рассинхронизировать flow-ы.

## Как работает

### RED
1. Считает **avg queue size** (EWMA от instantaneous).
2. Параметры **min_th, max_th, max_p**.
3. Логика drop:
   - avg < min_th → не drop.
   - avg > max_th → drop всё.
   - min_th ≤ avg ≤ max_th → drop с вероятностью, линейно растущей от 0 до max_p.

```
P(drop)
1.0 ─────┐
         │
max_p ─┐ │
       │ │
   0 ──┴─┴──── avg queue
       min  max
```

**Эффект:** TCP-flows получают сигнал перегрузки **постепенно**, разные в разные моменты → нет глобальной синхронизации.

### CoDel (Controlled Delay)
Современная альтернатива RED:
- Не считает queue size — считает **sojourn time** (сколько пакет сидит в очереди).
- Если sojourn > target (например, 5 мс) дольше interval (например, 100 мс) — drop один пакет.
- Cycle ускоряется при упорной перегрузке.

**fq_codel** = CoDel + per-flow fair queuing → ещё лучше для смешанного трафика.

## Пример
**Linux router:**
```bash
tc qdisc add dev eth0 root fq_codel
```
По умолчанию fq_codel в современных дистрибутивах. Bufferbloat в потребительских роутерах решается включением AQM.

**Дата-центры (low-latency):** часто используют **DCTCP** (DataCenter TCP) с ECN-маркировкой при превышении queue threshold — близко к AQM с маркировкой вместо drop'а.

## Связи
- **Базируется на:** [[Перегрузка сети]] (инструмент); EWMA-сглаживание для RED.
- **Используется в:** **CoDel/fq_codel** (Linux default), **PIE** (DOCSIS), **DCTCP** (DC).
- **Соседи по уровню:** [[ECN]] — маркировка вместо drop'а; обычно RED+ECN — лучшая комбинация.
- **Противопоставляется:** **drop tail** — проще, но даёт synchronization и bufferbloat.

## Подводные камни
- **RED tuning** сложен: 4 параметра, оптимум зависит от RTT и числа потоков. CoDel почти не нуждается в tuning.
- AQM **не работает**, если буфер слишком мал — пакеты дропаются до того, как AQM вмешается.
- **TCP-friendly** = AQM настроен так, чтобы TCP воспринимал drop как сигнал, не катастрофу. На UDP-flows AQM может быть жёстче.

## Дальше читать
- [[ECN]] — следующий шаг (маркировка вместо drop).
- [[TCP]] — главный потребитель сигналов AQM.
- Tanenbaum, гл. 5, §5.3.2 (стр. PDF 460–463).
