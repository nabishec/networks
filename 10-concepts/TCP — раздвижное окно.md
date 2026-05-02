---
title: TCP — раздвижное окно
aliases: ["TCP sliding window", "rwnd", "Nagle", "Silly window syndrome"]
type: concept
layer: transport
chapter: 6
difficulty: intermediate
prerequisites: ["[[TCP]]", "[[Скользящее окно]]"]
related: ["[[Управление потоком vs контроль перегрузки]]", "[[Bandwidth-Delay Product]]"]
tags: [networking, ch06]
---
# TCP — раздвижное окно (sliding window)

## TL;DR
Реализация [[Скользящее окно]] в TCP. Получатель в каждом ACK сообщает свой **rwnd** (Window-поле, 16 бит, до 64 КБ; с **Window Scale** до 1 ГБ). Отправитель **не отправляет больше rwnd unacked байт**. Дополнительно — congestion window (cwnd), и эффективное окно = `min(rwnd, cwnd)`. Также: алгоритм Нэгла, defense от Silly Window Syndrome.

## Какую проблему решает
Слать данные так быстро, как может медленный приёмник, — переполнение его буфера. Слать так медленно, как может — недогрузка сети. Sliding window даёт ровно то, что нужно: получатель сам говорит, сколько может ещё принять.

## Как работает

**Receive Window (rwnd):**
- Получатель имеет input buffer.
- В каждом ACK он шлёт `Window = свободное место в буфере`.
- Отправитель: `bytes_in_flight ≤ rwnd`.
- Если приложение получателя медленно читает → буфер заполняется → rwnd → 0 → отправитель блокируется (zero window).
- Получатель потом шлёт **window update** (ACK с большим rwnd).

**Window Scale (RFC 1323):**
- Поле Window 16 бит → max 64 КБ.
- На LFN (Long Fat Networks) этого мало.
- Опция в SYN: `window_scale = N` → реальное окно = Window << N.
- Max N=14 → 1 ГБ.
- **Обязательна** для современного high-BDP TCP.

**Алгоритм Нэгла (RFC 896):**
- Маленькие записи (1 байт telnet) → много мелких сегментов → ужасно эффективно.
- Nagle: «не шли маленький сегмент, если есть unacked данные. Жди ACK или MSS байт payload».
- Сглаживает silly small writes.
- **Отключается** через `TCP_NODELAY` для интерактивных приложений (REST API, BookSync).

**Silly Window Syndrome (Clark, RFC 1122):**
- Получатель шлёт rwnd=1 байт → отправитель шлёт 1-байтовый сегмент → получатель снова rwnd=1 → лавина мелких сегментов.
- Решение: получатель не объявляет rwnd до min(MSS, half buffer).

**Delayed ACK:**
- Получатель ждёт ~200 мс перед посылкой ACK — надеясь на piggyback с обратными данными.
- На bulk transfer без обратного трафика — лишняя задержка.

## Пример
**Скачивание файла по TCP:**
- Сервер отправляет, держа `bytes_in_flight ≤ rwnd`.
- Клиент: ОС читает быстро, rwnd ≈ buffer size (64 КБ * scale).
- При пересылке через медиум-RTT (50 мс) канал использован полностью пока buffer не упрётся в BDP.

**Zero window:** клиент разгрузил процессор → буфер опустошился → window update → отправитель продолжает.

## Связи
- **Базируется на:** [[Скользящее окно]] (общий принцип), [[TCP — заголовок]] (поле Window).
- **Используется в:** все TCP-передачи; [[Bandwidth-Delay Product]] (rwnd ≥ BDP).
- **Соседи по уровню:** [[TCP — slow start]], [[TCP — congestion avoidance]] (cwnd — congestion control).
- **Противопоставляется:** Stop-and-Wait — окно 1, неэффективно на BDP.

## Подводные камни
- **rwnd=0** не означает «соединение умерло» — буфер занят. Не путать с RST.
- **Без Window Scale** на LFN — недоутилизация. Все современные ОС включают по умолчанию.
- **Nagle + Delayed ACK** — известная неудачная пара: latency до 200 мс на интерактивных операциях. `TCP_NODELAY` или `TCP_QUICKACK`.
- **rwnd vs cwnd:** не путай! см. [[Управление потоком vs контроль перегрузки]].

## Дальше читать
- [[TCP — slow start]], [[TCP — congestion avoidance]] — congestion-side.
- [[Управление потоком vs контроль перегрузки]] — общая картина.
- Tanenbaum, гл. 6, §6.5.8 (стр. PDF 636–640).
