---
title: CUBIC TCP
aliases: ["CUBIC", "TCP CUBIC"]
type: algorithm
layer: transport
chapter: 6
difficulty: intermediate
prerequisites: ["[[TCP — congestion avoidance]]", "[[AIMD]]"]
related: ["[[BBR]]", "[[TCP — slow start]]"]
tags: [networking, ch06]
---
# CUBIC TCP (RFC 8312, 2018)

## TL;DR
**Default congestion control в Linux** с 2006 г. (BIC до 2.6.13, CUBIC с 2.6.19), стандартизован RFC 8312 (2018), обновлён **RFC 9438 (2023)** с уточнёнными константами. После потери cwnd срезается на **β=0.7** (не /2). Затем **кубический** рост: ближе к точке потери — медленно (mild-probe), дальше — быстро. Лучше AIMD на **high-BDP** сетях. RTT-fairness против Reno и multi-flow performance.

## Какую проблему решает
[[AIMD]] (Reno) на современных high-BDP сетях восстанавливается медленно: cwnd /= 2, линейный рост, много RTT. На канале 10 Гбит/с с RTT 100 мс одна потеря → ~minute восстановления до полной утилизации.

CUBIC: **держит** cwnd близко к точке последней потери (Wmax) → быстрее восстанавливается + менее агрессивный probe.

## Как работает

**Формула** (упрощённо):
$$ W(t) = C \cdot (t - K)^3 + W_{\max} $$

где:
- W_max — cwnd на момент последней потери.
- K — время от потери, когда cwnd должен достичь W_max снова: `K = ∛(W_max·(1−β)/C)`.
- t — время с момента потери.
- C ≈ 0.4 (масштабный коэффициент).

**Поведение:**
- Сразу после потери — медленный рост (concave region, чтобы не вызвать новую потерю).
- Достижение W_max в момент K — горизонтальный «плато».
- Дальше — быстрый рост (convex region) для probe нового максимума.

```
cwnd
W_max ──────────────╱
       ╲           ╱
        ╲         ╱  ← CUBIC: kubic curve
         ╲       ╱
   W_max·β       │
               t = K
```

**По сравнению с Reno:**
- Reno: linear+slow восстановление.
- CUBIC: концентрируется около Wmax → лучшее использование канала.

**β=0.7** (вместо 1/2 в Reno): меньшее срезание при потерях.

## Пример
- **Linux на сервере 2010+:** sysctl `net.ipv4.tcp_congestion_control = cubic` (default).
- Скачивание большого файла через 100 Мбит/с-канал с потерями 0.01% → CUBIC даёт ~95% утилизации; Reno — ~70%.

**Замена:** в современных Linux дистрибутивах часто включён [[BBR]] (Google) для исходящего трафика серверов.

## Связи
- **Базируется на:** [[TCP — congestion avoidance]] (модификация AI), [[TCP — slow start]] (стартовая фаза та же).
- **Используется в:** Linux default (с 2006), Windows 10+ (опционально), macOS Big Sur+ опционально.
- **Соседи по уровню:** [[BBR]] — model-based альтернатива; **PRR** (Proportional Rate Reduction) — компонент.
- **Противопоставляется:** **TCP Reno/New Reno** — линейный рост; CUBIC — кубический.

## Подводные камни
- CUBIC всё ещё **loss-based** — на каналах с **случайными** (не congestion) потерями (Wi-Fi, mobile) даёт ложные срезания. BBR справляется лучше.
- **RTT-fairness** между CUBIC-потоками лучше, чем у Reno (Reno дискриминирует long-RTT flows). Между CUBIC и BBR на одном канале — отдельная история (BBR обычно «забирает» больше).
- На LFN с buffer'ами CUBIC может **наполнить очереди** до потери → bufferbloat.

## Дальше читать
- [[BBR]] — современный конкурент.
- [[TCP — congestion avoidance]] — родитель.
- Tanenbaum, гл. 6, §6.5.11 (стр. PDF 654).
