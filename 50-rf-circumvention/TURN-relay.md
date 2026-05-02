---
title: TURN-relay
aliases: ["TURN", "coturn"]
type: concept
layer: application
status: working
status_as_of: 2026-05-02
risk: low
prerequisites: ["[[VoIP и SIP]]", "[[NAT]]"]
related: ["[[WebRTC-туннель]]"]
sources: [src-01, src-07]
tags: [networking, rf-circumvention, concept]
---
# TURN-relay (Traversal Using Relays around NAT, RFC 5766)

## TL;DR
**TURN-сервер** — relay для WebRTC/SIP-трафика, когда **прямой peer-to-peer невозможен** (двойной NAT, симметричный NAT). Клиент A шлёт TURN-серверу → TURN пересылает клиенту B (и обратно). В контексте обхода блокировок — TURN превращается в **VPN-relay**: клиент шлёт TURN-серверу, тот шлёт в Internet. **coturn** — главная open-source реализация.

## Какую проблему решает
**Обычный WebRTC** делает peer-to-peer; TURN срабатывает когда P2P не работает.

**В контексте РФ-обхода:**
- TURN-сервер выглядит как **легитимный WebRTC-relay** (Zoom использует TURN, Discord, etc.).
- DPI не блокирует TURN, потому что это сломало бы массу видеозвонков.
- Используется как **transport** для VPN-трафика (см. [[WebRTC-туннель]]).

## Как работает

**Stun vs TURN:**
- **STUN** (RFC 5389) — узнать свой публичный IP/port; для P2P-discovery.
- **TURN** — relay-сервер, который **пересылает** трафик.

**TURN-протокол:**
- UDP (порт 3478, иногда 5349 для TLS).
- **Allocate** request: клиент просит TURN выделить relay-port.
- **Send** indication: «отправь данные X на peer-IP».
- TURN пересылает.

**Defaults coturn:**
```
listening-port=3478
tls-listening-port=5349
relay-ip=YOUR_PUBLIC_IP
realm=your-domain.com
user=username:password
cert=/path/to/cert.pem
pkey=/path/to/key.pem
```

## Где ломается / почему может не работать
- **TURN over TLS** на 5349 — если IP-blacklist, не пройдёт.
- **Bandwidth-cost:** relay через TURN — серверу нужно обрабатывать **двойной** трафик (входящий + исходящий).
- **TURN-публикация:** часть бесплатных TURN-серверов (twilio, openrelay) могут быть в blacklist.
- **Latency:** TURN всегда добавляет один hop по сравнению с P2P.

## Минимальный пошаговый сценарий
**coturn на VPS:**
```bash
sudo apt install coturn
sudo nano /etc/turnserver.conf
# Минимум:
# listening-port=3478
# realm=my.example.com
# user=test:test
# fingerprint
# lt-cred-mech

sudo systemctl start coturn
```

**Проверка:**
```bash
turnutils_uclient -t -u test -w test 1.2.3.4
```

## Что нужно
- VPS с public IP.
- coturn (open-source).
- Опционально: TLS-сертификат (Let's Encrypt) для TURN-S.

## Связи
- **Базируется на:** [[VoIP и SIP]] (концепт relay для медиа), [[NAT]] (главное применение — NAT traversal).
- **Используется в:** [[WebRTC-туннель]], real-WebRTC-приложения (Zoom, Discord).
- **Соседи по уровню:** **STUN** (для discovery), ICE (algorithm выбора пути).

## Источники
- src-01, src-07.
