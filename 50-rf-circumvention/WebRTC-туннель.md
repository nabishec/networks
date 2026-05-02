---
title: WebRTC-туннель
aliases: ["olcRTC", "OlcRTC", "RTC Tunnel", "WebRTC-обход"]
type: technique
layer: application
status: working
status_as_of: 2026-05-02
risk: medium
prerequisites: ["[[VoIP и SIP]]", "[[RTP]]", "[[Туннелирование]]"]
related: ["[[TURN-relay]]", "[[CDN-фронтинг]]"]
sources: [src-01, src-03, src-07]
tags: [networking, rf-circumvention, technique]
---
# WebRTC-туннель

## TL;DR
Туннелирование данных через **WebRTC-соединение** (как в видеозвонке Zoom/Discord/Telegram). Трафик упакован в **DTLS-SRTP** поверх UDP, идёт через **STUN/TURN-relay** или peer-to-peer. DPI видит «обычный видеозвонок» — блокировать = ломать все видеоконференции. Реализации: **olcRTC** (туннель через подделку Discord/Zoom-call), **RTC Tunnel**, **AmneziaWG**-flavors.

## Какую проблему решает
- DPI **не блокирует WebRTC**, потому что это сломало бы Zoom/Teams/Discord/Telegram/whatsapp-calls.
- WebRTC живёт **поверх UDP** — обходит TCP-замораживание.
- Trafic выглядит как **видеоконференция**, не VPN.
- Подходит для **whitelist-моделей**: WebRTC-сервисы обычно в whitelist (Zoom, Discord используются массово).

## Как работает

**Архитектура:**
```mermaid
flowchart LR
  Client -->|"DTLS-SRTP / WebRTC"| TURN[TURN-server<br/>(coturn / commercial)]
  TURN -->|"внутренний backend"| Exit[Экзит-VPN-сервер]
  Exit --> Internet
```

**Технические детали:**
- Клиент устанавливает WebRTC PeerConnection с TURN-сервером.
- Через **DataChannel** WebRTC (бинарный канал, не media) шлёт VPN-payload.
- TURN-server переадресует на backend.
- Encryption: **DTLS** (Datagram TLS).

**olcRTC** (src-01) — конкретная реализация: эмулирует pattern Discord-call. Связь устанавливается через signaling-сервер; data идёт по WebRTC DataChannel.

## Где ломается / почему может не работать
- **STUN/TURN-IP** должны быть в whitelist (если whitelist жёсткий).
- **Bandwidth:** WebRTC оптимизирован для real-time медиа, не bulk transfer — throughput ниже TCP-VPN.
- **Latency overhead:** DTLS-handshake + ICE-negotiation добавляет ~500 мс к старту.
- **DPI-эволюция:** ML-детектор может отличить «реальный звонок» (motion + audio patterns) от «VPN-пакетов».
- **Mobile-batterydrain:** WebRTC-stack тратит больше энергии, чем простой VPN.

## Минимальный пошаговый сценарий
1. Поднять **TURN-server** (coturn) на VPS вне whitelist (или внутри для cascade).
2. Настроить **signaling-server** (websocket для negotiation).
3. На клиенте — WebRTC-клиент (browser-based или native), который создаёт PeerConnection и пускает payload в DataChannel.
4. На backend — обратная сторона: принимает WebRTC, разворачивает payload, шлёт в Internet.

## Что нужно
- VPS под TURN (coturn — open-source).
- Signaling (Node.js + ws / Python aiohttp).
- Клиент с WebRTC (браузер, mobile-app, или native через libwebrtc).
- Опционально: реальный домен для STUN с TLS.

## Связи
- **Базируется на:** [[VoIP и SIP]] (родственная медиа-архитектура), [[RTP]] (DTLS-SRTP — родственник), [[Туннелирование]] (концепт).
- **Соседи по уровню:** [[TURN-relay]] (компонент), [[CDN-фронтинг]] (другая школа).
- **Используется в:** [[PB3 — 4-уровневая архитектура за 265₽]] (как fallback), olcRTC.
- **Противопоставляется:** TCP-based VPN (легче блокируются точечно).

## Источники
- src-01, src-03, src-07.
