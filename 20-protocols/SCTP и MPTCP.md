---
title: SCTP и MPTCP
aliases: ["SCTP", "MPTCP", "Multipath TCP", "Будущее TCP"]
type: protocol
layer: transport
chapter: 6
difficulty: advanced
prerequisites: ["[[TCP]]"]
related: ["[[QUIC]]", "[[Производительность LFN]]"]
tags: [networking, ch06]
---
# SCTP и MPTCP — расширения TCP-семантики

## TL;DR
**SCTP** (Stream Control Transmission Protocol, RFC 9260) — альтернатива TCP с **сохранением границ сообщений**, **multiple streams** (без HoL blocking) и **multi-homing** (один endpoint = несколько IP). Изначально для signaling в SS7; не стал mainstream в интернете. **MPTCP** (Multipath TCP, RFC 8684) — расширение TCP, использующее **несколько subflows** одновременно (Wi-Fi + cellular). iOS использует с 2013 для Siri, Apple Music. Tanenbaum §6.6.3 — «будущее TCP».

## Какую проблему решает
Классический TCP — **single-stream, single-path**. Это:
- HoL-blocking при потерях (HTTP/2 пытается решить через streams, но ловится TCP-уровнем).
- Один путь — нет redundancy / aggregation (Wi-Fi+LTE одновременно).
- Не сохраняет message boundaries (приложение само должно).

SCTP/MPTCP/QUIC — разные подходы к расширению.

## Как работает

### SCTP

**Главные особенности:**
- **Chunks** в сегменте — несколько msg в одном packet.
- **Message-oriented:** sender shлёт «cobtain» — receiver получает целиком (как UDP), но reliable (как TCP).
- **Multiple streams** в одном association — потеря в stream A не блокирует stream B.
- **Multi-homing:** endpoint имеет ≥1 IP — failover на резервный путь автоматически.
- **4-way handshake** (защита от SYN flood через INIT-cookie).
- Используется: SS7 over IP (Diameter), WebRTC data channel в браузерах.

**Не стал mainstream**: middleboxes (NAT, firewall) не понимают SCTP → пакеты дропаются. Аналогично всему non-TCP/UDP-трафику.

### MPTCP (Multipath TCP)

**Идея:** TCP-соединение состоит из **нескольких subflows**, каждый — отдельный TCP по разному пути.
- Например: Wi-Fi-subflow + LTE-subflow.
- Аппликация видит **один сокет** — MPTCP-стек балансирует payload между subflows.
- **Seamless handover:** Wi-Fi-сигнал слабеет → MPTCP перенаправляет в LTE без разрыва соединения.

**Использование:**
- **iPhone Siri** (2013+): voice через MPTCP — Wi-Fi + LTE. Нет паузы при выходе из дома.
- **Apple Music streaming.**
- **Корп. SDN/SD-WAN** иногда.
- **Linux MPTCPv1** mainlined в 5.6 (2020).

**Совместимость:**
- В SYN — MPTCP-option.
- Если peer не поддерживает — fallback на single TCP.
- Middleboxes частично уступают — сложно deploy.

## Пример
**iPhone в метро:**
- Дома Wi-Fi → metro → пропадает Wi-Fi → автоматически LTE.
- Без MPTCP: Siri-разговор обрывается (TCP связь привязан к IP).
- С MPTCP: subflow Wi-Fi умирает; subflow LTE продолжает; голос не рвётся.

## Связи
- **Базируется на:** [[TCP]] (расширение или альтернатива), [[Three-way handshake]] (с modifications).
- **Используется в:** Apple ecosystem (MPTCP), VoIP-телефония SS7-over-IP (SCTP), WebRTC data channels (SCTP).
- **Соседи по уровню:** [[QUIC]] — другой modern transport, **выиграл**, потому что в user-space (быстрая итерация). MPTCP/SCTP в kernel — дольше deploy.
- **Противопоставляется:** plain TCP — single-path, message-stream-only.

## Подводные камни
- **Middlebox-hostility** — главный killer SCTP и MPTCP в публичном интернете.
- **MPTCP scheduling complexity** — какой пакет в какой subflow. Round-robin? RTT-based? Простые scheduler'ы могут терять throughput.
- **MPTCP security:** добавляет attack surface (token-based hijacking без MPTCP-token).
- **QUIC's connection migration** в чём-то перекрывает MPTCP, но не точно — QUIC migrates one path, не использует одновременно много.

## Дальше читать
- [[QUIC]] — победитель в современном мире transport.
- [[TCP]] — родительская технология.
- Tanenbaum, гл. 6, §6.6.3 (стр. PDF 658).
