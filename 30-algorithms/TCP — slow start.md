---
title: TCP — slow start
aliases: ["Slow start", "Initial cwnd"]
type: algorithm
layer: transport
chapter: 6
difficulty: intermediate
prerequisites: ["[[TCP]]", "[[AIMD]]"]
related: ["[[TCP — congestion avoidance]]", "[[CUBIC TCP]]", "[[BBR]]"]
tags: [networking, ch06]
---
# TCP — slow start (Van Jacobson, 1988)

## TL;DR
Фаза начала TCP-соединения: cwnd начинается **малым** (initial window 1–10 MSS) и **удваивается каждый RTT**. Несмотря на название, это **самая быстрая** фаза роста — экспоненциальная. Длится до достижения **ssthresh** (slow start threshold) или потери; затем переход в [[TCP — congestion avoidance]] с линейным ростом.

## Какую проблему решает
В 1986 г. интернет реально коллапсировал из-за congestion collapse — TCP без ограничений утоплял сеть. Van Jacobson предложил: **начинать осторожно** и **постепенно** ускоряться, давая сети время адаптироваться. Это и есть slow start.

## Как работает

**Алгоритм:**
1. **Начало:** cwnd = initial window (исторически 1 MSS; современно — RFC 6928, до 10 MSS).
2. **Каждый ACK:** cwnd += 1 MSS.
3. **За один RTT** все ACK'и приходят → cwnd удваивается.
4. **Останавливается, когда:**
   - cwnd ≥ ssthresh → переход в congestion avoidance (линейный рост).
   - **Timeout (RTO):** cwnd сбрасывается до **1 MSS** (или IW), ssthresh = cwnd/2; снова slow start с нуля.
   - **3 dup-ACK** (fast retransmit, RFC 5681): cwnd ← ssthresh = max(FlightSize/2, 2·MSS); **не** возврат в slow start, а fast recovery → congestion avoidance.
   - Достигнут rwnd получателя → ограничение flow control.

```
cwnd
  16 ─┐                       ╱
      │                      ╱  ← linear (congestion avoidance)
   8 ─│                  ╱──╱
      │              ╱──╱
   4 ─│        ╱──╱──╱  ← exponential (slow start)
   2 ─│   ╱──╱
   1 ─└──╱─────────────── time (RTT)
        1   2   3   4   5
```

**Причина названия «slow»:** на момент изобретения **обычные** TCP начинали с большого cwnd → коллапсы. «Slow» = осторожно. По сравнению с linear growth (CA) slow start экспоненциально **быстрый**.

**Initial Window** (IW):
- RFC 2581 (1999): IW=1 MSS.
- RFC 3390: IW = 2-4 MSS в зависимости от размера.
- **RFC 6928 (2013): IW = 10 MSS** (~14 КБ) — стандарт сегодня.

## Пример
**Сервер начинает HTTP-ответ 100 КБ:**
- IW=10 MSS (~14 КБ).
- RTT = 50 мс.
- Через 1 RTT: cwnd=20.
- Через 2 RTT: cwnd=40.
- Через 3 RTT (~150 мс) — cwnd=80, отправляли уже 80 MSS = ~115 КБ → файл уйдёт.

Без IW=10 (если IW=1) занял бы ~7 RTT — заметно медленнее на коротких HTTP-ответах. Поэтому IW увеличивали.

## Связи
- **Базируется на:** [[TCP]], [[AIMD]] (slow start — фаза до AI).
- **Используется в:** все TCP-implementations; стартовая фаза CUBIC, Reno, New Reno.
- **Соседи по уровню:** [[TCP — congestion avoidance]] (линейная фаза после ssthresh).
- **Противопоставляется:** [[BBR]] не использует традиционный slow start — у него своя startup-фаза с измерением BW.

## Подводные камни
- **На коротких потоках** (HTTP-ответ < BDP) slow start не успевает разогнаться → недоутилизация. **Initial Window** увеличивали именно из-за этого.
- **После idle period** многие реализации сбрасывают cwnd обратно в IW (RFC 5681) — RTO_idle. Поэтому keep-alive с регулярным трафиком быстрее, чем сессии с паузами.
- **Slow start может «overshoot» ёмкость**: когда cwnd удвоился до значения за пределом BW*RTT — куча пакетов в очереди → потери. **Hybrid Slow Start (HyStart)** в Linux пытается выявить «коленку» и перейти раньше.

## Дальше читать
- [[TCP — congestion avoidance]] — фаза после ssthresh.
- [[CUBIC TCP]], [[BBR]] — современные альтернативы.
- Tanenbaum, гл. 6, §6.5.10 (стр. PDF 643–654).
