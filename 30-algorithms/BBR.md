---
title: BBR
aliases: ["Bottleneck Bandwidth and RTT", "BBRv1", "BBRv2", "BBRv3"]
type: algorithm
layer: transport
chapter: 6
difficulty: advanced
prerequisites: ["[[TCP — congestion avoidance]]", "[[Bandwidth-Delay Product]]"]
related: ["[[CUBIC TCP]]", "[[Bufferbloat]]"]
tags: [networking, ch06]
---
# BBR — Bottleneck Bandwidth and RTT (Google, 2016)

## TL;DR
BBR — современный алгоритм, **не нагружающий сеть** так же, как старые. Аналогия: едешь по шоссе. **Старые TCP (Reno/CUBIC)**: «едь быстрее, пока не догонишь бампер впереди» — они ждут потерь как сигнала. **BBR**: «измеряй знаки и поток, держи безопасную скорость». Меньше потерь, меньше задержек — особенно на Wi-Fi и мобильных, где random-loss есть всегда. Включён в Google для YouTube/Search; в Linux с ядра 4.9 (2016). **BBRv3** (2023, Linux 6.x) — ECN-aware и лучшее co-existence с CUBIC.

## Какую проблему решает
Loss-based congestion control (Reno/CUBIC) **создаёт** перегрузку, чтобы её обнаружить: «толкать пока не дропнут». Это:
- Заполняет буферы → bufferbloat → большие RTT.
- Реагирует на **любые** потери, в том числе случайные на Wi-Fi/mobile.

BBR смотрит на **measurement-based model** канала: какая ёмкость, какая задержка? Строит rate точно под BDP, без переполнения.

## Как работает

**Состояние BBR:**
- **BtlBw** — оценка bottleneck bandwidth (max_bw, наблюдаемая в окне).
- **RTprop** — оценка propagation RTT (min observed).
- BDP_est = BtlBw × RTprop.
- cwnd = ~2 × BDP_est (для headroom).

**Pacing:** BBR не просто шлёт «всё что в cwnd», а **раскладывает** во времени с rate ≈ BtlBw → не создаёт burst'ов в буферах.

**Фазы:**
1. **STARTUP:** экспоненциальный рост (как slow start), пока BW не перестанет расти 3 RTT подряд.
2. **DRAIN:** очистить переполнение буфера.
3. **PROBE_BW:** периодический probe вверх (8 фаз, ~1.25× BW в одной из них) и вниз (~0.75×) — для отслеживания изменений ёмкости.
4. **PROBE_RTT:** раз в ~10 сек cwnd снижается до 4 пакетов на 200 мс — обновить RTprop.

**Версии:**
- **BBRv1** (2016): агрессивен в сосуществовании с CUBIC (получает больше), плохо реагирует на загруженные buffers.
- **BBRv2** (2019): добавлены reaction на потери, ECN, fairness.
- **BBRv3** (2023): дальнейшие улучшения, скоро mainline.

## Пример
**YouTube использует BBR для исходящего видео:**
- На лоссистом mobile-канале CUBIC падал на каждом random loss → CUBIC throughput плохой.
- BBR не реагирует на random loss → стабильно держит ~BDP → throughput выше.
- В A/B-тесте Google: рост QoE от BBR заметный.

**Linux включить:**
```bash
sysctl net.ipv4.tcp_congestion_control=bbr
```

## Связи
- **Базируется на:** [[Bandwidth-Delay Product]] (BBR измеряет именно его), [[TCP]] / QUIC (BBR работает и в QUIC).
- **Используется в:** Google services (YouTube, GFE), всё больше в production Linux-серверах, QUIC-стеки.
- **Соседи по уровню:** [[CUBIC TCP]] (loss-based соперник), L4S — другая современная парадигма с ECN.
- **Противопоставляется:** **loss-based** family (Reno, CUBIC). BBR — measurement-based.

## Подводные камни
- **Coexistence с CUBIC:** BBRv1 «забирает» долю у CUBIC на shared bottleneck. BBRv2/v3 решают.
- **Не учитывает ECN** в v1. В v2/v3 добавили.
- **Probe RTT** — раз в 10 с резкое падение cwnd до минимума → видимый glitch на видеоконференциях. Tunable.
- **Pacing** требует точного timing'а от ОС → лучше работает с накладным TC qdisc'ом fq.

## Дальше читать
- [[CUBIC TCP]] — loss-based соперник.
- [[Bufferbloat]] — то, что BBR смягчает.
- Tanenbaum, гл. 6, §6.6.2 (стр. PDF 655–658).
