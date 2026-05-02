---
title: XTLS-Vision
aliases: ["XTLS-RPRX-Vision", "Vision flow"]
type: technique
layer: application
status: working
status_as_of: 2026-05-02
risk: medium
prerequisites: ["[[TLS — рукопожатие]]", "[[VLESS-Reality]]"]
related: ["[[xHTTP]]"]
sources: [src-02, src-08]
tags: [networking, rf-circumvention, technique]
---
# XTLS-Vision (flow)

## TL;DR
**Flow** для VLESS, устраняющий «TLS-в-TLS-маскировку», характерную для классических TLS-прокси. Внутри туннеля шифрование делается **только для первого трафика handshake'а** + критических кадров; после установления — payload идёт напрямую через outer TLS без второго слоя. Это **снижает CPU**, делает поведение пакетов **похожим на обычный HTTPS** (та же длина и timing), и убирает аномалию double-TLS, которую DPI могут заметить.

## Какую проблему решает
**TLS-in-TLS** — классическая аномалия классических VPN (OpenVPN-TLS, classical Trojan): inner TLS внутри outer TLS даёт характерный паттерн (record-в-record, double-handshake-times). DPI с ML-детекцией ловит. Vision: только outer-TLS видим; inner — без TLS-преамбулы.

## Как работает

**Структура потока:**
1. Client+Server TLS handshake (outer) → cert от target (если Reality).
2. Vision negotiates flow=xtls-rprx-vision.
3. Sender шифрует **handshake-данные** клиента (~первые KB) обычным TLS-record'ом.
4. После critical-frame: **direct copy** payload в TLS-record без extra-encryption (TLS-уровень и так шифрует).

**Параметр Xray-конфига:**
```json
"flow": "xtls-rprx-vision"
```

## Где ломается / почему может не работать
- Не работает с **TCP-обёртками без TLS** (Shadowsocks, plain VLESS) — нужен TLS поверх.
- При **xHTTP packet-up** — Vision не нужен (сам xHTTP меняет shape трафика).
- В **РФ-2026** Vision рекомендуется в паре с Reality для устранения NewSessionTicket-fingerprint.

## Минимальный пошаговый сценарий
В JSON-конфиге Xray (server и client):
```json
"settings": { "clients": [{ "id": "...", "flow": "xtls-rprx-vision" }] }
"streamSettings": { "security": "reality", "realitySettings": { ... } }
```

## Что нужно
- Xray-core ≥ v25.x.
- TLS-stream (Reality или классический).

## Связи
- **Базируется на:** [[TLS — рукопожатие]] (outer TLS).
- **Используется в:** [[VLESS-Reality]] (типичная пара), [[PB7 — basic VLESS-Reality с нуля]].
- **Соседи по уровню:** [[xHTTP]] (альтернативная transport-обёртка), uTLS.
- **Противопоставляется:** classical VLESS+TLS без flow — TLS-in-TLS видна DPI.

## Источники
- src-02, src-08.
