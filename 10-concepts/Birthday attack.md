---
title: Birthday attack
aliases: ["Атака дней рождения", "Birthday paradox"]
type: concept
layer: security
chapter: 8
difficulty: basic
prerequisites: ["[[Хеш-функции]]"]
related: ["[[Цифровая подпись]]"]
tags: [networking, ch08]
---
# Birthday attack

## TL;DR
В hash из N бит **collision** (два разных input с одинаковым output) находится за **~2^(N/2)** попыток, не 2^N. Это «парадокс дней рождения»: в группе из 23 человек вероятность совпадения дней рождения > 50%, хотя дней 365. Поэтому **128-битные hash не secure** для collision-resistance: 2^64 attempts feasible (Bitcoin делает столько за минуты). 256+ бит обязательны.

## Какую проблему решает
Объяснить, почему **длина hash output должна быть в 2 раза больше**, чем требуемая security level. SHA-1 (160 бит) → 80-бит security → сегодня сломано (SHAttered, 2017). SHA-256 → 128-бит → пока safe.

## Как работает

**Парадокс дней рождения:**
- Дней в году: 365.
- Сколько людей нужно, чтобы вероятность совпадения > 50%? **23**.
- Не 365/2 ≈ 182, как интуиция!

Причина: пар людей растёт квадратично. С 23 людьми — `23·22/2 = 253` пары → достаточно для probability > 0.5.

**Применение к hash:**
- Hash output N бит = 2^N возможных values.
- Collision: H(M1) = H(M2) для M1 ≠ M2.
- При случайных M-ах — **expected collision after ~ √(2^N) = 2^(N/2)** trials.

**Hash sizes vs security:**

| Hash size | Pre-image | Collision (birthday) |
|---|---|---|
| 64 bit | 2^64 | 2^32 — trivial |
| 128 bit | 2^128 | 2^64 — feasible (Bitcoin-scale) |
| 256 bit | 2^256 | 2^128 — astronomically infeasible |

Поэтому для 128-bit security нужно **256-bit hash**.

## Пример
**Атака на сертификат:**
- Атакующий хочет, чтобы CA подписал legitimate `cert1`, и при этом существовал `cert2` с тем же hash.
- Тогда подпись от cert1 = подпись от cert2 → forge.
- С MD5 (128 бит) это возможно (2^64) — было сделано в 2008 (rogue CA via MD5 collisions).
- С SHA-256 (256 бит) — нет.

**Bitcoin block hash collisions:**
- Hash блока 256 бит. Birthday → 2^128. Не feasible.
- Если бы 128 бит → 2^64 ≈ 2 хэша/сек × год выполнило бы — **broken**.

## Связи
- **Базируется на:** теория вероятности (combinatorics), [[Хеш-функции]].
- **Используется в:** определение security-bound для hash sizes; design choices для signatures.
- **Соседи по уровню:** **rainbow tables** — другая атака на hash, для passwords.
- **Противопоставляется:** **brute-force pre-image** (full 2^N) — сложнее.

## Подводные камни
- **128 бит security ≠ 128-bit hash output.** Для 128-bit security нужен 256-bit hash.
- **Quantum threat:** Grover algorithm reduces brute force from 2^N к 2^(N/2). Combined with birthday: hash N бит → ~2^(N/3) collision quantum. Поэтому SHA-512 рассматривается для post-quantum margin.
- **Collision vs second pre-image:** разные свойства. Birthday attack — на collision. Second pre-image требует 2^N в любом случае.

## Дальше читать
- [[Хеш-функции]] — где это применяется.
- [[Цифровая подпись]] — потребитель.
- Tanenbaum, гл. 8, §8.7.4 (стр. PDF 886–888).
