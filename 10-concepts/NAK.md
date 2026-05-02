---
title: NAK
aliases: ["Negative Acknowledgement", "NACK"]
type: concept
layer: data-link
chapter: 3
difficulty: basic
prerequisites: ["[[ARQ]]", "[[Selective Repeat]]"]
related: ["[[Go-Back-N]]"]
tags: [networking, ch03]
---
# NAK — Negative Acknowledgement

## TL;DR
**Отрицательное подтверждение**: получатель явно сообщает «я **не получил** seq=N» — отправителю не нужно ждать timeout, чтобы сделать retransmit. Используется в [[Selective Repeat]]-протоколах, в SACK-блоках TCP, в RTP/RTCP-feedback. Ускоряет recovery от потерь по сравнению с timer-only.

## Какую проблему решает
В чистом ARQ (только positive ACK) отправитель узнаёт о потере **по таймауту**: если ACK не пришёл за RTO — повторяем. NAK ускоряет: получатель **сам говорит** о потере, как только обнаружил (gap в seq numbers). Особенно важно в RT-приложениях (видео): timer-based слишком медленно.

## Как работает

**Сценарий с NAK:**
1. Получатель ждёт seq=10.
2. Приходит seq=12 (gap → seq 11 потерян).
3. Получатель шлёт **NAK(11)**.
4. Отправитель видит NAK → retransmit seq=11 **немедленно**.

**Без NAK** отправитель ждал бы RTO (100-200 мс), потом обычно через **3 dup-ACK** в TCP — 3 RTT.

**Tanenbaum в §3.4.2** (Protocol 6 — Selective Repeat) даёт детальную механику с переменной `no_nak`:
- NAK шлётся только **раз** на каждый missing seq (защита от дубликатов NAK).
- `no_nak = false` пока ACK на этот seq не пришёл.
- После retransmit и ACK — `no_nak = true`, дальнейшие out-of-order не вызывают новых NAK.
- **Auxiliary `ack_timer`** для случая, когда NAK потерян.

## Пример
**Selective Repeat** на satellite-канале:
- RTT 600 мс — timer-only retransmit очень медленный.
- NAK detection при out-of-order delivery — отправитель повторяет за 0.5 RTT.

**RTP/RTCP NACK** для real-time:
- В видеоконференции получатель видит «потерян frame N» → шлёт NACK.
- Отправитель повторяет (если не слишком поздно).
- Иначе — request keyframe (полный intra-frame для recovery).

**TCP SACK** (Selective ACK, RFC 2018):
- Не точно NAK, но семантически похоже: получатель сообщает «у меня пришли блоки [a..b], [c..d]» → отправитель видит дыры.
- Положительное reporting вместо отрицательного, но тот же эффект.

## Связи
- **Базируется на:** [[ARQ]] (общий принцип), [[Selective Repeat]] (Protocol 6).
- **Используется в:** Satellite ARQ, RTP/RTCP, TCP SACK (концептуально).
- **Соседи по уровню:** **DUP-ACK** в TCP — позитивный аналог NAK (3 dup → fast retransmit).
- **Противопоставляется:** **timer-only retransmit** — медленнее на потерях.

## Подводные камни
- **NAK может потеряться** — нужен резервный timer (ack_timer в Tanenbaum'е).
- **Дубликаты NAK** делают retransmit fights — нужен flag `no_nak` после первого NAK на seq.
- **NAK** означает «retry me» — не подходит для real-time, где старые данные бесполезны (в видео старый frame не нужен, лучше keyframe).
- В **multicast** NAK создаёт **NAK implosion**: тысячи получателей шлют NAK одному источнику. Решение — **suppression** (random delay перед NAK + listen, если кто-то уже шлёт).

## Дальше читать
- [[Selective Repeat]] — главное применение.
- [[ARQ]] — общий контекст.
- Tanenbaum, гл. 3, §3.4.2 (стр. PDF 286–296).
