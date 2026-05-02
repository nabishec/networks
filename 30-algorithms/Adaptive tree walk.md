---
title: Adaptive tree walk
aliases: ["Adaptive tree walk protocol", "Capetanakis"]
type: algorithm
layer: mac
chapter: 4
difficulty: intermediate
prerequisites: ["[[Проблема распределения канала]]", "[[ALOHA]]"]
related: ["[[Bit-map и token-passing]]", "[[Binary countdown]]"]
tags: [networking, ch04]
---
# Adaptive tree walk (Capetanakis, 1979)

## TL;DR
Семейство **limited-contention** MAC-протоколов, превращающих коллизии в **двоичный поиск** виновников. После коллизии узлы делятся на 2 половины — каждая пробует слот. Конфликт в одной → ещё деление. Без конфликтов в двух — продолжаем с следующей группой. **O(log N)** разрешение коллизии. Подходит при **умеренной** нагрузке (между ALOHA и token-passing).

## Какую проблему решает
[[ALOHA]] хорош при низкой нагрузке. [[Bit-map и token-passing]] хороши при высокой. Между ними — узкая полоса оптимума. Limited-contention протоколы (LC, MLMA, adaptive tree walk) **адаптивны**: ведут себя как ALOHA при low-load, как bitmap при high-load.

## Как работает

**Дерево станций:**
- N станций — листья двоичного дерева log₂(N) глубины.
- При **collision** — обходим дерево.

**Алгоритм:**
1. Начинаем с **корня** — все N станций пытаются в одном слоте.
2. Если успех — следующий слот, корень снова.
3. Если коллизия — спускаемся в **левое поддерево** (станции с битом-индекса 0).
4. Если успех / пусто — переходим к **правому поддереву**.
5. Если ещё коллизия в левом — спускаемся в его поддерево.
6. Полное обходание → возвращаемся к корню для следующего цикла.

**Адаптация:**
- При **низкой загрузке**: начинаем с уровня **0** (корень — все станции).
- При **высокой**: начинаем с уровня **i** = log₂(q) — оптимально, где q — оценка числа активных.

**Performance:**
- Низкая нагрузка: ~63% utilization (близко к slotted ALOHA + meliorations).
- Умеренная: 0.46 (vs 0.37 slotted ALOHA).
- Высокая: gradually deg, но никогда не collapses как ALOHA.

## Пример
**Раннее VSAT** (Very Small Aperture Terminal) спутниковые сети 1980-90-х использовали similar contention-resolution — много terminals shared transponder.

В **современных** сетях редко напрямую. Но идея «binary backoff на дереве станций» вошла в **CSMA/CD binary exponential backoff** (упрощённая) и в **TDMA contention-resolution** в DOCSIS.

## Связи
- **Базируется на:** [[Проблема распределения канала]], идеи tree-search в комбинаторике.
- **Используется в:** raznych research-LC protocols, WiMAX-style ranging, DOCSIS upstream contention.
- **Соседи по уровню:** [[Binary countdown]] (другой LC), [[ALOHA]] (просчищает в low-load).
- **Противопоставляется:** [[Bit-map и token-passing]] — fully scheduled, no contention.

## Подводные камни
- **Требует synchronized slots** — slot times.
- **Time-synchronized clocks** между станциями.
- **Учебная** ценность выше практической в 2026 г. Прорыв CSMA/CD + switched Ethernet вытеснил LC.
- В **Bluetooth piconet** (master polling) — концептуально близко, но centralized.

## Дальше читать
- [[Bit-map и token-passing]] — alternative scheduling.
- [[Binary countdown]] — соседний LC.
- Tanenbaum, гл. 4, §4.2.4 (стр. PDF 329–333).
