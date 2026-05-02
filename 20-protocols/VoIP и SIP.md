---
title: VoIP и SIP
aliases: ["VoIP", "SIP", "Voice over IP"]
type: protocol
layer: application
chapter: 7
difficulty: basic
prerequisites: ["[[RTP]]", "[[UDP]]"]
related: ["[[Цифровой звук — кодеки]]", "[[Цифровое видео — кодеки]]"]
tags: [networking, ch07]
---
# VoIP и SIP

## TL;DR
**VoIP** (Voice over IP) — голосовая связь поверх интернета. **Аудио** идёт по [[RTP]] поверх UDP с кодеком (Opus, G.711). **Сигнализация** (звонок, ответ, BYE) — через **SIP** (Session Initiation Protocol, RFC 3261), HTTP-подобный текстовый протокол. WebRTC — современный набор: SIP-like signaling + ICE/STUN/TURN для NAT-traversal + DTLS-SRTP для шифрования.

## Какую проблему решает
Передача голоса по интернету дешевле и гибче PSTN. До VoIP — отдельная телефонная инфраструктура. С VoIP — голос как любые данные интернета: международные звонки бесплатно, intеграция с приложениями, видеоконференции.

## Как работает

**Двухканальная архитектура:**
1. **Signaling** (SIP) — установление, разрыв, capabilities exchange.
2. **Media** (RTP/SRTP) — собственно голос/видео.

**SIP — methods:**
- **INVITE** — инициирует звонок, в SDP описаны media-параметры.
- **TRYING (100), RINGING (180), OK (200)** — provisional/final responses.
- **ACK** — подтверждение OK.
- **BYE** — разрыв.
- **REGISTER** — telegist у proxy.

**Типичный flow:**
```
A → Proxy → B
A: INVITE B (SDP: I support Opus, my IP=...)
B → A: 180 RINGING
B → A: 200 OK (SDP: I support Opus, my IP=...)
A → B: ACK
A ↔ B: RTP audio (peer-to-peer if NAT allows)
A: BYE
B: 200 OK
```

**SDP** (Session Description Protocol, RFC 4566) — описание медиа-сессии в SIP-payload (кодеки, RTP-порты, capabilities).

**NAT-traversal:**
- **STUN** — узнать свой публичный IP/port.
- **TURN** — relay-сервер, если direct невозможен.
- **ICE** — алгоритм выбора лучшего пути из кандидатов.

**WebRTC:**
- Standard для browser-based real-time.
- Не SIP, но похожее signaling (custom через WebSocket обычно).
- Media: SRTP (encrypted RTP).
- Включён в Chrome/Firefox/Safari → видеоконференции в браузере без plugins.

## Пример
**Звонок через Skype/Teams:**
- При логине: REGISTER на signaling-сервер.
- При звонке: INVITE через сервер (он мостит peer-to-peer media-paths).
- Голос: Opus 24 кбит/с в SRTP.
- Если NAT: TURN-сервер relay'ит media.
- BYE при отбое.

**SIP-trunk:** компания подключается напрямую к VoIP-провайдеру через SIP. Прощай PSTN-CPE.

## Связи
- **Базируется на:** [[RTP]] (media), [[UDP]] (transport обычно), [[Цифровой звук — кодеки]] (Opus, G.711).
- **Используется в:** Skype, Teams, Zoom, Discord, WebRTC; mobile VoLTE; corporate phone systems.
- **Соседи по уровню:** WebRTC — современный стек; **XMPP/Jingle** — open альтернатива.
- **Противопоставляется:** PSTN — выделенная сеть; VoIP — over-the-top.

## Подводные камни
- **NAT** — главная боль VoIP. Без STUN/TURN — звонок не работает.
- **Latency requirements:** для естественной беседы <150 мс one-way. На дальних маршрутах + кодек buffer = борьба за миллисекунды.
- **Quality of Service:** в DiffServ-ной сети голос обычно EF (Expedited Forwarding) → priority queue. На публичном интернете гарантий нет.
- **Encryption:** в современном VoIP всегда SRTP+DTLS; old SIP без TLS читался провайдером.

## Дальше читать
- [[RTP]] — media transport.
- [[Цифровой звук — кодеки]] — Opus, G.711.
- Tanenbaum, гл. 7, §7.4.4 (стр. PDF 774–785).
