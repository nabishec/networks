---
title: TCP fast path и header prediction
aliases: ["Fast path TCP", "Header prediction", "Van Jacobson optimization"]
type: algorithm
layer: transport
chapter: 6
difficulty: advanced
prerequisites: ["[[TCP]]", "[[TCP — заголовок]]"]
related: ["[[TCP — состояния]]", "[[Производительность LFN]]"]
tags: [networking, ch06]
---
# TCP fast path и header prediction (Van Jacobson, 1989)

## TL;DR
Главная производительная оптимизация в TCP-стеке: для **типичного случая** (ESTABLISHED + ожидаемый seq + полный сегмент + нет URG/RST) вся обработка делается коротким **fast path** без проверки всех условий протокола. Шаблон-заголовок («header prediction template») сравнивается с пришедшим — совпало → fast path; нет — full slow path. Hit rate ~90% в нормальной сети. Базовая техника всех современных high-performance TCP-стеков.

## Какую проблему решает
TCP-обработка пакета — десятки проверок (state machine, options, sequence wraparound, edge cases). На 100+ Гбит/с-сетях каждая инструкция CPU дорога. Van Jacobson заметил: **большинство пакетов в установившемся соединении** — обычные «следующий ожидаемый seq, full data segment, ESTABLISHED». Для них не нужно проверять всё — достаточно проверить, что **header выглядит как ожидается**.

## Как работает

**Шаблон prediction (для receiver):**
- Состояние = ESTABLISHED.
- Flags = ACK only (no SYN/FIN/RST/URG).
- seq = expected (next byte we wait).
- ack-num = что ожидаемое ACK от sender.
- TCP options length = 0 (или только timestamp).
- Length matches MSS.

**Сравнение через single XOR/CMP:** хост хранит «prediction header» в **прототипе** — XOR его с приходящим. Если все «importantt» поля совпадают (XOR == 0 для них) — fast path; иначе — slow path.

**Fast path actions (несколько инструкций):**
1. Скопировать payload в user buffer (с pre-computed checksum).
2. Обновить expected seq.
3. Шлёт ACK (если delayed timer expired или enough data).
4. **Готово.**

**Slow path:** полная state-machine, options parsing, edge case handling.

**Hit rate:** ~90% в нормальной сети (steady-state bulk transfer). Драма случается только на edge'ах (handshake, FIN, retransmit).

## Доп. оптимизации Van Jacobson + Clark (1989)

1. **Combining checksum + copy** — вычислять checksum **во время** копирования из NIC-buffer в user-buffer. Один проход вместо двух.
2. **Hash-table connections lookup** + last-used pointer (90% hit на самом частом).
3. **Header compression** для LFN ([[Сжатие заголовков]]) — meta-оптимизация.

## Пример
**Linux kernel TCP fast path:**
- В `tcp_v4_rcv()` — проверка: ESTABLISHED + simple → `tcp_rcv_established()` fast path.
- Иначе — `tcp_rcv_state_process()` slow path с full state machine.
- На современных серверах процессинг пакета — **сотни ns** (не μs).

**Современное:**
- **TSO** (TCP Segmentation Offload) на NIC — segmentation в железе.
- **GRO** (Generic Receive Offload) — собирает фрагменты для batched fast path.
- **DPDK / kernel bypass** — обходят kernel вообще, custom TCP в user-space.
- **eBPF** — programmable fast paths.

## Связи
- **Базируется на:** [[TCP]] (что оптимизируется), [[TCP — заголовок]] (что сравнивается).
- **Используется в:** все production-TCP стеки (Linux, BSD, Windows, mTCP, Snabb).
- **Соседи по уровню:** **TSO/GRO/LRO** — hardware offloads; **kernel bypass** (DPDK) — radical-fast-path.
- **Противопоставляется:** «полная state machine на каждом пакете» — медленно для bulk transfer.

## Подводные камни
- **Header prediction misbehaves** при reordering/loss — частый slow path.
- **TSO + congestion control** — иногда вводит в заблуждение CC (видит «один сегмент» вместо реальных N).
- На **shallow buffers** или high-loss каналах fast-path-rate низкий → CPU-cost растёт.
- В **container/multitenant** environments per-flow optimizations теряют эффективность.

## Дальше читать
- [[TCP — заголовок]] — что matches в шаблоне.
- [[Производительность LFN]] — родственный контекст.
- Tanenbaum, гл. 6, §6.7.6 (стр. PDF 666–669).
