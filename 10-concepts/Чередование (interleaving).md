---
title: Чередование (interleaving)
aliases: ["Interleaving", "Чередование"]
type: concept
layer: data-link
chapter: 3
difficulty: basic
prerequisites: ["[[Корректирующие коды — Хэмминг]]"]
related: ["[[Код Рида-Соломона]]", "[[LDPC и сверточные коды]]"]
tags: [networking, ch03]
---
# Чередование (interleaving)

## TL;DR
Приём, превращающий **single-error-correcting код** в **burst-error-tolerant** через перетасовку битов перед передачей. Биты разных кодовых слов **перемешиваются** во времени. На приёме — обратная перетасовка. Burst-ошибка распределяется по разным codewords, каждый получает только 1-2 ошибки → SECC справляется.

## Какую проблему решает
Простые коды (Хэмминг (7,4)) исправляют **одну** ошибку в блоке. Burst-ошибка длиной 5 бит → 5 ошибок в одном codeword → не исправить. Без interleaving такие burst'ы (царапина на CD, помеха в радиоэфире) уничтожали бы коммуникацию.

Interleaving не добавляет избыточности — просто перемешивает существующее. Дёшево, эффективно.

## Как работает

**Block interleaving:**
- Имеется N codewords длины K = блок N×K.
- Записываем построчно (codeword за codeword).
- Передаём **по столбцам**.
- Burst в столбце N бит → 1 ошибка в каждом из N codewords (а не N в одном).

```
Coded:    [c1: ABCDEFG][c2: HIJKLMN][c3: OPQRSTU][c4: VWXYZ12]
Matrix (4×7):
A B C D E F G
H I J K L M N
O P Q R S T U
V W X Y Z 1 2

Read column-wise: A H O V B I P W C J Q X ...
Interleaved:    A H O V B I P W C J Q X D K R Y ...

If burst loses positions 5-9 (B I P W C):
- c1 loses B → 1 error
- c2 loses I → 1 error
- c3 loses P → 1 error
- c4 loses W → 1 error

Каждый codeword: 1 error → SECC fixes.
```

**Convolutional interleaving:** более эффективное по памяти; использует sliding-window-style permutations.

## Пример
- **CD-Audio:** **CIRC** (Cross-Interleaved Reed-Solomon Code) — 2 уровня RS + interleaving. Восстанавливает царапины до 2.5 мм.
- **DVB-T (terrestrial digital TV):** RS(204, 188) + convolutional interleaver.
- **GSM:** convolutional code + interleaver — burst во время fast fading'а распределяется.
- **LTE:** turbo coding + interleaving.
- **DSL** (G.992): block-interleaver добавляет latency но повышает BER.

## Связи
- **Базируется на:** [[Корректирующие коды — Хэмминг]] / [[Код Рида-Соломона]] / [[LDPC и сверточные коды]] (любой single/few-error код).
- **Используется в:** все wireless-стандарты, optical disks, satellite.
- **Соседи по уровню:** **scrambling** — другая перетасовка, но для устранения long runs of zeros (синхронизация), не для burst protection.
- **Противопоставляется:** «без interleaving» — burst убивает codeword.

## Подводные камни
- **Latency:** chunk N codewords надо собрать перед отправкой → deinterleaving на приёме также требует buffer'а. На VoIP/RT — критично.
- **Trade-off depth:** глубокое interleaving (большой блок) → лучше protection vs больших burst'ов, но больше latency.
- В **современном LDPC + AWGN-channel** interleaving не критичен (random errors), но всё ещё используется в bursty channels.

## Дальше читать
- [[Корректирующие коды — Хэмминг]], [[Код Рида-Соломона]] — базовые коды.
- Tanenbaum, гл. 3, §3.2.2 (стр. PDF 260).
