---
title: coturn
aliases: ["coturn", "TURN-сервер", "STUN/TURN-сервер"]
type: tool
layer: application
status: working
status_as_of: 2026-05-02
risk: low
prerequisites: ["[[TURN-relay]]", "[[NAT]]"]
related: ["[[WebRTC-туннель]]", "[[TURN-relay]]"]
sources: [src-07]
tags: [networking, rf-circumvention, tool]
---
# coturn

## TL;DR
**Открытая reference-реализация STUN/TURN-сервера** на C (`coturn-project/coturn`, GitHub). Реализует RFC 5389 (STUN), RFC 5766 (TURN), RFC 6062 (TCP-allocations), RFC 8656 (TURN/UDP/TCP/TLS/DTLS). Стандарт de facto для self-hosted сигналинга в Matrix-Synapse, Jitsi, Nextcloud Talk, кастомных WebRTC-приложениях. В сценариях обхода РФ-блокировок используется как **relay-точка для [[WebRTC-туннель|WebRTC-туннелей]]** — DTLS-SRTP-канал к coturn выглядит как обычный видеозвонок и часто проходит сквозь whitelist-фильтр (см. src-07).

## Что делает
- **STUN-сервис:** клиенту отвечает «вот твой публичный IP/порт после [[NAT]]» — для прямого peer-to-peer.
- **TURN-relay:** если direct connection невозможен (CGNAT, симметричный NAT, fw) — coturn принимает медиа-поток от клиента-A и пересылает клиенту-B; оба соединяются с coturn, не друг с другом.
- **Authentication:** long-term credentials (username/password из БД) или **REST-API auth** с TTL-токенами.
- **Транспорты:** UDP / TCP / TLS / DTLS — 3478 (plain), 5349 (TLS).

## Где взять
- Repo: <https://github.com/coturn/coturn>
- Debian/Ubuntu: `apt install coturn`
- Docker: множество image'ов, `instrumentisto/coturn` популярный.

## Использование (минимум)

`/etc/turnserver.conf`:
```
listening-port=3478
tls-listening-port=5349
realm=example.com
fingerprint
lt-cred-mech
user=alice:secret
cert=/etc/letsencrypt/live/example.com/fullchain.pem
pkey=/etc/letsencrypt/live/example.com/privkey.pem
no-loopback-peers
no-multicast-peers
```
Запустить: `systemctl enable --now coturn`. Открыть UDP/TCP 3478 и 5349 на firewall'е, пробросить в DNAT при необходимости.

## РФ-сценарий обхода
- coturn ставится на **российском VPS** (cheap, Yandex/VK Cloud); WebRTC-клиент устанавливает DTLS-SRTP-сессию через coturn → выходит на удалённый peer.
- Видимый трафик — UDP 3478/5349 с DTLS-обёрткой; для DPI это «звонок».
- Связки: [[WebRTC-туннель]] + coturn → реализует «олРТЦ»-туннель (см. src-01) и связанные сценарии в [[PB3 — 4-уровневая архитектура за 265₽]], [[PB6 — Nginx+LE с разделением IP]].

## Связи
- **Базируется на:** [[TURN-relay]] (концепт), [[NAT]] (для STUN-traversal).
- **Используется в:** [[WebRTC-туннель]], Matrix Synapse, Jitsi-Meet, Nextcloud Talk, custom WebRTC-приложения.
- **Соседи по уровню:** **Pion** (Go-WebRTC-stack, своё libpion-server), **eturnal** (Erlang TURN от ProcessOne).
- **Противопоставляется:** **публичные TURN-сервисы** (Twilio, Xirsys) — платные и не контролируешь exit-IP; coturn self-hosted — больше контроля.

## Подводные камни
- **Authentication обязательна** — открытый TURN превращается в open-relay (DDoS-amplifier, чужой трафик через ваш IP).
- В РФ-2025+ TURN-серверы на «не-whitelist»-AS могут блокироваться по IP-reputation; cтавить coturn на **whitelist-AS** (Yandex/VK Cloud) — практичнее.
- coturn **жрёт CPU** на TLS — для крупных нагрузок нужен offload или горизонтальное масштабирование.
- Reuse-IP для STUN и TURN на одном порту 3478 — типичная конфигурация; разделять порты для отладки.
- Уязвимости были (CVE-2020-26262 — обход allowed-peer); держать в актуальных версиях.

## Источники
- src-07 — Soldier22, Habr 1013122 (TURN/coturn в whitelist-обходе).
- Habr: [TURN/STUN — платить или крутить свой?](https://habr.com/ru/articles/923862/) — обзор self-hosted vs SaaS.
- Habr: [Развёртывание узла Matrix-Synapse, Coturn и Element](https://habr.com/ru/companies/selectel/articles/943150/) — практический setup.
- Repo: <https://github.com/coturn/coturn>
