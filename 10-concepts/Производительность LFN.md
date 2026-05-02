---
title: Производительность LFN
aliases: ["Long Fat Networks", "LFN", "Производительность сетей с большой пропускной способностью"]
type: concept
layer: transport
chapter: 6
difficulty: intermediate
prerequisites: ["[[Bandwidth-Delay Product]]", "[[TCP]]"]
related: ["[[CUBIC TCP]]", "[[BBR]]", "[[Сжатие заголовков]]"]
tags: [networking, ch06]
---
# Производительность LFN (Long Fat Networks)

## TL;DR
**LFN** = «**длинная толстая сеть**»: высокая пропускная способность × большой RTT → большой [[Bandwidth-Delay Product|BDP]]. Например, 10 Гбит/с × 100 мс = 125 МБ. На LFN классический TCP плохо себя ведёт: окно мало, восстановление после потерь долгое, накладные расходы стэка велики. Современные техники: **window scaling**, **SACK**, **TCP timestamps**, **CUBIC/BBR**, **TSO/GRO**, **kernel bypass** (DPDK, XDP).

## Какую проблему решает
В дата-центрах и магистралях скорости приближаются к 100/400 Гбит/с, RTT межконтинентальные — сотни мс. На таких каналах:
- Один TCP-flow может **не утилизировать** канал.
- Каждый перерыв (потеря, idle) дорого обходится.
- CPU стэка становится bottleneck, не сеть.

## Как работает

**Проблемы LFN и решения:**

| Проблема | Решение |
|---|---|
| Window 64 КБ < BDP | **Window Scale** (RFC 1323) — до 1 ГБ |
| Cumulative ACK теряет точность | **SACK** (RFC 2018) — selective ACK |
| Старые сегменты приняты как новые (большие seq cycle) | **PAWS** (Protect Against Wrapped Sequences) с timestamps |
| Долгое восстановление после потери (Reno) | [[CUBIC TCP]] / [[BBR]] |
| CPU per-packet overhead | **TSO** (TCP Segmentation Offload) — NIC сегментирует; **GRO** (Generic Receive Offload) — собирает |
| Slow start IW=1 на коротких потоках | **IW=10** (RFC 6928) |
| Random losses в Wi-Fi/mobile дают AIMD-falls | [[BBR]] — не loss-based |
| Header overhead на маленьких пакетах | [[Сжатие заголовков]] |
| Kernel context-switches | **kernel bypass**: DPDK, XDP, AF_XDP, io_uring |
| AF между несколькими flows | MPTCP (Multipath TCP) — параллельно по нескольким интерфейсам |

**Sysctl tuning в Linux:**
```bash
# Window auto-tuning
net.ipv4.tcp_rmem = "4096 87380 67108864"   # max 64 МБ
net.ipv4.tcp_wmem = "4096 65536 67108864"

# CUBIC default; BBR опционально
net.ipv4.tcp_congestion_control = bbr

# Initial cwnd (RFC 6928)
ip route change ... initcwnd 10
```

## Пример
**Скачивание 1 ГБ файла США → Россия (RTT 100 мс):**
- Канал 1 Гбит/с.
- BDP = 12.5 МБ.
- Без window scale: rwnd ≤ 64 КБ → effective ~6 Мбит/с → 22 мин.
- С window scale + CUBIC: ~10 мин (упирается в потери далеко не всё).
- С BBR + 0 потерь: ~9 мин (ближе к канальной скорости).

**Дата-центр 100 Гбит/с RDMA:** TCP+ядро не справляется → **InfiniBand** или **RoCE** (RDMA over Converged Ethernet) с kernel-bypass.

## Связи
- **Базируется на:** [[Bandwidth-Delay Product]] (определение LFN), [[TCP]] (главный страдалец).
- **Используется в:** [[CUBIC TCP]], [[BBR]], [[QUIC]], [[Сжатие заголовков]] — все для LFN.
- **Соседи по уровню:** **kernel bypass** (DPDK/XDP), **MPTCP**, **QUIC** — современные подходы.
- **Противопоставляется:** «обычный TCP без tuning» — на LFN недоутилизирует канал.

## Подводные камни
- **Window auto-tuning** в Linux работает хорошо, но требует достаточно RAM на сервере (memory ↔ buffer).
- **TSO/GRO** скрывают realный размер сегмента от congestion control → на потерях поведение может удивить.
- **MPTCP** требует поддержки на обеих сторонах + middlebox-friendly options.
- На скорости 100+ Гбит/с **TCP сам по себе** становится bottleneck — индустрия уходит на QUIC/RDMA/специализированные стеки.

## Дальше читать
- [[Bandwidth-Delay Product]] — фундамент.
- [[CUBIC TCP]], [[BBR]] — TCP-optimization.
- Tanenbaum, гл. 6, §6.7 (стр. PDF 658–676).
