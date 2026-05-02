---
title: WebSocket
aliases: ["WebSocket", "WS", "WSS", "RFC 6455"]
type: term
layer: application
chapter: 7
difficulty: basic
prerequisites: ["[[HTTP]]"]
related: ["[[HTTP-2 и HTTP-3|HTTP/2 и HTTP/3]]", "[[TLS — рукопожатие]]", "[[Динамические веб-приложения]]"]
tags: [networking, ch07, glossary]
---
# WebSocket (RFC 6455)

## TL;DR
**Двунаправленный фреймовый канал** поверх одного TCP-соединения, открываемого через HTTP-Upgrade. Клиент шлёт обычный HTTP-запрос с `Connection: Upgrade`, `Upgrade: websocket`; сервер отвечает `101 Switching Protocols`, и поверх той же TCP-сессии стороны обмениваются **бинарными или текстовыми фреймами** (frame = opcode + length + payload). URL-схемы: `ws://` (порт 80, plain) и `wss://` (порт 443, поверх [[TLS — рукопожатие|TLS]]). RFC 6455 (2011). Идеальна для чатов, real-time-уведомлений, online-игр, биржевых тикеров.

## Зачем нужен
HTTP/1.1 — request-response, push с сервера невозможен из коробки (long-polling и SSE — частичные обходы). Для real-time-приложений нужен **двунаправленный** канал, который браузер может открыть напрямую, без плагинов и proprietary-протоколов. WebSocket пробрасывается через корпоративные firewalls и HTTPS-proxy «как HTTPS», работает в любом современном браузере.

## Что подтверждает книга
В Tanenbaum 6e отдельной главы про WebSocket нет — материал ниже основан на RFC 6455 и обзорах с Habr (см. источники). В книге обсуждаются HTTP, persistent connections и архитектура динамических веб-приложений (гл. 7, §7.3.3), но не WebSocket конкретно.

## Как работает (минимум)

**Handshake (HTTP Upgrade):**
```
GET /chat HTTP/1.1
Host: example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
```

**Сервер отвечает:**
```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

`Sec-WebSocket-Accept` = base64(SHA-1(key + GUID)). После этого ответа TCP-сокет уже не HTTP — стороны шлют **WebSocket-фреймы**.

**Frame (упрощённо):** opcode (text/binary/close/ping/pong) + длина + payload (с маскированием от клиента).

## Связи
- **Базируется на:** [[HTTP]] (handshake), [[TLS — рукопожатие]] (для `wss://`), TCP.
- **Используется в:** real-time чаты (Slack, Discord), live-уведомления, онлайн-игры, торговые терминалы, signaling в WebRTC.
- **Соседи по уровню:** **SSE** (Server-Sent Events) — однонаправленный server→client поверх HTTP/1.1; **HTTP/2 server push** (deprecated 2022); **WebTransport** на QUIC — современный преемник.
- **Противопоставляется:** REST/HTTP-polling — по сути «опрос» с overhead; gRPC bidirectional streaming — то же на HTTP/2.

## Подводные камни
- **Нет встроенной аутентификации** — только cookie/токен в первом HTTP-handshake. Дальше канал «доверенный» по факту установления.
- **Балансировщики L4** не понимают upgrade — нужна sticky-session или L7-aware-LB (HAProxy в режиме `mode http`, NGINX `proxy_http_version 1.1` + `Upgrade`).
- **Нет автоматических reconnect** — клиентскому коду нужно обрабатывать обрывы (heartbeat ping/pong, восстановление состояния).
- **HTTP/2** имеет «WebSockets over HTTP/2» (RFC 8441), но поддержка пятнистая — много прокси «расщепляют» upgrade.
- **WebSocket-туннели** иногда применяются как транспорт для прокси-сервисов; в РФ-DPI могут отслеживать аномалии (постоянная активность, длинная сессия) — близко к [[Session freezing]].

## Дальше читать
- [[HTTP]] — базовый протокол.
- [[HTTP-2 и HTTP-3|HTTP/2 и HTTP/3]] — современные транспорты, частично перекрывающие use-cases.
- Habr: [Анатомия WebSocket: человечный разбор RFC 6455](https://habr.com/ru/articles/1004772/) — пошаговый разбор.
- Habr: [WebSocket: разбираем как работает](https://habr.com/ru/sandbox/171066/) — handshake и фреймы.
- Habr: [WebSocket для начинающих системных аналитиков: просто о сложном. Часть 1](https://habr.com/ru/articles/886802/) — обзор для нетехнической аудитории.
- RFC 6455 — официальная спецификация.

## Источники
- Habr: <https://habr.com/ru/articles/1004772/>, <https://habr.com/ru/sandbox/171066/>, <https://habr.com/ru/articles/886802/>.
