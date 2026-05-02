---
title: AIMD
aliases: ["Additive Increase Multiplicative Decrease"]
type: algorithm
layer: transport
chapter: 6
difficulty: intermediate
prerequisites: ["[[Управление потоком vs контроль перегрузки]]"]
related: ["[[TCP — slow start]]", "[[TCP — congestion avoidance]]", "[[CUBIC TCP]]"]
tags: [networking, ch06]
---
# AIMD — Additive Increase, Multiplicative Decrease

## TL;DR
Базовый принцип TCP-congestion control (Chiu-Jain, 1989): при отсутствии потерь — **линейный рост** cwnd (`+1 MSS` за RTT); при потере — **умножение на коэффициент <1** (`cwnd /= 2`). Из всех пар (увеличение, уменьшение) только AIMD сходится к **fair + efficient** состоянию: разные потоки выравниваются по доле канала, и сумма не превышает ёмкости.

## Какую проблему решает
Несколько потоков делят узкое место. Хотим: (1) не перегружать (efficient — не зашкаливать); (2) делить ёмкость честно (fair — равные доли). Без координации: как «договариваются»? AIMD — простое правило, локально применяемое каждым потоком, дающее глобальные свойства fairness и stability.

## Как работает

**Геометрический анализ (Chiu-Jain):**
- Состояние сети — точка (x₁, x₂) — окна двух потоков.
- Линия эффективности: x₁+x₂ = C (ёмкость).
- Линия fairness: x₁ = x₂.
- AIMD-движение:
  - **AI** — точка идёт по диагонали +ε,+ε → уходит от fairness, но к перегрузке.
  - **MD** — умножение на k<1 — точка приближается к fairness (по линии в начало координат).
- Циклы AI/MD дают **сходимость к идеальной точке** на пересечении линий.

**В TCP:**
- AI: при удачном RTT cwnd += MSS² / cwnd ≈ +1 MSS за RTT.
- MD: при потере cwnd /= 2.

```
cwnd
 ╱╲    ╱╲    ╱╲
╱  ╲  ╱  ╲  ╱
╲   ╲╱   ╲╱
 ←AI→ MD ←AI→ MD
```

«Pilот зуба» — характерная форма cwnd во времени.

**Альтернативные пары:**
- AIAD (additive both): не сходится к fairness.
- MIMD (multiplicative both): осцилляции.
- MIAD: расходится.
- **AIMD** — единственный сходится.

## Пример
- TCP Reno (1990) — каноническое AIMD.
- TCP New Reno, SACK — улучшения детекции потерь, но AIMD-ядро прежнее.
- **CUBIC** ([[CUBIC TCP]], 2008) — модифицирует AI: вместо линейного роста — кубический. Лучше для high-BDP.
- **BBR** ([[BBR]]) — отказывается от AIMD, на потерях не реагирует, model-based.

## Связи
- **Базируется на:** [[Управление потоком vs контроль перегрузки]] (это — congestion алгоритм), [[Перегрузка сети]] (на что реагирует).
- **Используется в:** TCP Reno/New Reno, ECN-aware TCP; в модифицированной форме — CUBIC.
- **Соседи по уровню:** [[TCP — slow start]] (фаза до AIMD), [[TCP — congestion avoidance]] (это AIMD после ssthresh).
- **Противопоставляется:** AIAD/MIMD/MIAD — не работают; [[BBR]] — не loss-based, отказ от AIMD-парадигмы.

## Подводные камни
- AIMD **уважает потери** как сигнал. На каналах с **случайными** потерями (Wi-Fi, мобильные) даёт ложные снижения cwnd → недоутилизация. Решения — RED+ECN, BBR.
- Fairness AIMD — **per-flow**. Атакующий с N параллельными flow получает N долей.
- **Convergence time:** на high-BDP сетях AIMD-восстановление после потери занимает много RTT → CUBIC лучше.

## Дальше читать
- [[TCP — slow start]], [[TCP — congestion avoidance]] — фазы TCP.
- [[CUBIC TCP]], [[BBR]] — современные эволюции.
- Tanenbaum, гл. 6, §6.3.1 (стр. PDF 599–604).
