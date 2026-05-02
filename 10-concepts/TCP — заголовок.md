---
title: TCP — заголовок
aliases: ["TCP header", "TCP-segment"]
type: concept
layer: transport
chapter: 6
difficulty: intermediate
prerequisites: ["[[TCP]]"]
related: ["[[TCP — состояния]]", "[[TCP — раздвижное окно]]"]
tags: [networking, ch06]
---
# TCP — заголовок (RFC 9293)

## TL;DR
**Минимум 20 байт**, до 60 с опциями. Поля: src/dst port (16+16), sequence (32), ack (32), data offset (4), **8 флагов** (CWR/ECE/URG/ACK/PSH/RST/SYN/FIN — NS из RFC 3540 deprecated в RFC 9293), window (16), checksum (16), urgent (16), options. Каждое поле — рычаг управления: state machine, надёжность, поток, перегрузка.

## Какую проблему решает
Понимать формат TCP-заголовка нужно: для отладки (Wireshark), оптимизации (MSS, window scale), безопасности (RST attacks, SYN flood), и просто для понимания **как** TCP делает то, что делает.

## Как работает

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          Source Port          |       Destination Port        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                        Sequence Number                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Acknowledgement Number                     |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Data |       |C|E|U|A|P|R|S|F|                               |
| Offset| Reserv|W|C|R|C|S|S|Y|I|            Window             |
|       |       |R|E|G|K|H|T|N|N|                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|           Checksum            |         Urgent Pointer        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Options (если data offset > 5)              |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

**Поля:**
- **Sequence Number** — номер первого байта payload в потоке (или ISN для SYN).
- **Ack Number** — следующий ожидаемый seq. Cumulative ACK.
- **Data Offset** — длина заголовка в 32-битных словах (5 = 20 байт без options).
- **Флаги:**
  - **SYN/FIN** — открытие/закрытие.
  - **ACK** — поле Ack Number значимо.
  - **RST** — аварийный разрыв.
  - **PSH** — push (передать приложению немедленно).
  - **URG** — urgent pointer значим.
  - **ECE/CWR** — для ECN ([[ECN]]).
- **Window** — receive window (rwnd, для flow control).
- **Checksum** — pseudo-header + TCP-header + data.

**Ключевые опции (в options-секции):**
- **MSS** (Maximum Segment Size) — обмен в SYN.
- **Window Scale** (RFC 1323) — множитель к Window полю (rwnd до 1 ГБ).
- **SACK Permitted / SACK** (RFC 2018) — selective ACK.
- **Timestamps** — для PAWS, RTT-измерения.

## Пример
**Wireshark одной TCP-сессии:**
- SYN: seq=12345, ack=0, флаги [SYN], options [MSS=1460, SACK, Window scale=7].
- SYN-ACK: seq=98765, ack=12346, флаги [SYN, ACK].
- ACK: seq=12346, ack=98766, [ACK].
- Data: seq=12346, ack=98766, [PSH, ACK], payload «GET /».

## Связи
- **Базируется на:** [[TCP]] (структура его сегментов).
- **Используется в:** [[TCP — состояния]] (флаги управляют переходами), [[TCP — раздвижное окно]] (window).
- **Соседи по уровню:** [[Internet checksum]] (поле Checksum).
- **Противопоставляется:** UDP-заголовок — 8 байт без всех этих полей.

## Подводные камни
- **Cumulative ACK** не различает «потеряны 3 пакета» и «один потерян, остальные пришли». **SACK** решает.
- **Window field 16 бит = max 64 КБ** без window scale — давно мало для high-BDP сетей. Window Scale **обязателен** в современном TCP.
- **PSH** flag в современных стеках почти не имеет наблюдаемого эффекта — всё передаётся приложению быстро.

## Дальше читать
- [[TCP]] — общий контекст.
- [[TCP — состояния]] — флаги в действии.
- Tanenbaum, гл. 6, §6.5.4 (стр. PDF 628–631).
