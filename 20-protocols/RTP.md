---
title: RTP
aliases: ["Real-time Transport Protocol", "RTCP"]
type: protocol
layer: transport
chapter: 6
difficulty: intermediate
prerequisites: ["[[UDP]]"]
related: ["[[VoIP и SIP]]", "[[QUIC]]", "[[Multicast routing]]"]
tags: [networking, ch06]
---
# RTP — Real-time Transport Protocol (RFC 3550)

## TL;DR
Протокол поверх **UDP** для real-time медиа (голос, видео, потоковое аудио). Каждый пакет содержит **sequence number** (детектировать потери) и **timestamp** (синхронизация playback). Потери допустимы, retransmission бесполезна. Сопровождается **RTCP** — обратной связью о качестве (loss rate, jitter). Базис VoIP, видеоконференций (H.323, SIP, WebRTC).

## Какую проблему решает
TCP retransmit'ит потерянное → **поздний** пакет, не нужен в real-time (картинка/звук давно пройдены). UDP сам не дает sequence/timestamp — приложение должно. RTP стандартизует это.

## Как работает

**Заголовок RTP (12+ байт):**
```
 0                   1                   2                   3
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|V=2|P|X|  CC   |M|     PT      |       sequence number         |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                           timestamp                            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|           synchronization source (SSRC) identifier             |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|            contributing source (CSRC) identifiers (опц.)       |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

- **PT** — Payload Type (G.711, Opus, H.264 и т.п.).
- **Sequence number** — детектирование потерь.
- **Timestamp** — playback timing (часы кодека).
- **SSRC** — идентификатор потока (например, конкретный микрофон).

**RTCP (Real-time Transport Control Protocol):**
- Параллельный поток отчётности.
- **Sender Reports / Receiver Reports** — статистика loss/jitter.
- Источник может адаптировать кодек (понижать битрейт при потерях) на основе RTCP feedback.

**Buffering на приёме (de-jitter buffer):**
- Пакеты по UDP приходят с jitter'ом (вариация задержки).
- Приёмник буферизует ~30–100 мс → отдаёт кодеку равномерно.
- Большой буфер → меньше потерь, больше задержка.

## Пример
**VoIP-звонок:**
- Голос кодируется Opus → 20 мс блоки.
- Каждый блок — RTP-пакет (sequence++, timestamp+=160 при 8 кГц).
- RTCP каждые ~5 с шлёт RR/SR.
- Приёмник де-джиттерит 40 мс → плавный звук.

**WebRTC:** в браузере — RTP/RTCP над UDP (через ICE/STUN/TURN для NAT-traversal), DTLS-SRTP для шифрования.

## Связи
- **Базируется на:** [[UDP]] (транспорт), [[Internet checksum]] (защита).
- **Используется в:** [[VoIP и SIP]], WebRTC, видеоконференции, IPTV, Zoom/Meet/Teams.
- **Соседи по уровню:** **SRTP** (Secure RTP) — с шифрованием и аутентификацией; **WebRTC** — стек, использующий RTP.
- **Противопоставляется:** [[TCP]] — для real-time бесполезен (HoL blocking).

## Подводные камни
- RTP **не делает** ничего сам — он структура данных. Шифрование (SRTP), reliability (NACK extensions), congestion control (TWCC) — отдельные слои.
- **Тайминг кодека ≠ тайминг сети.** Timestamp основан на частоте дискретизации (8 кГц для G.711), не wall clock.
- В **NAT** RTP+RTCP идут на разных портах — нужен NAT-traversal с обоих.

## Дальше читать
- [[UDP]] — транспорт.
- [[VoIP и SIP]] — главный потребитель.
- Tanenbaum, гл. 6, §6.4.3 (стр. PDF 616–623).
