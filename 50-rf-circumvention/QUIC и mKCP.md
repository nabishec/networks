---
title: QUIC и mKCP — сравнение UDP-туннелей
aliases: ["QUIC tunnel", "mKCP-обзор", "Hysteria-сводка", "QUIC и mKCP"]
type: technique
layer: transport
status: partial
status_as_of: 2026-05-02
risk: medium
prerequisites: ["[[QUIC]]", "[[UDP]]"]
related: ["[[Hysteria-2]]", "[[mKCP]]", "[[VLESS-Reality]]", "[[Shadowsocks-2022]]", "[[AmneziaWG]]"]
sources: [src-08, src-06]
tags: [networking, rf-circumvention, technique]
---
# QUIC и mKCP — сравнение UDP-туннелей в РФ

> **Это сводная заметка.** Атомные детали — в [[Hysteria-2]] и [[mKCP]].

## TL;DR
Класс **UDP-based туннелей** обходит [[Session freezing]] (TCP-замораживание) и работает быстрее на lossy-radio. Главные имплементации:
- **[[Hysteria-2]]** — QUIC + Brutal-congestion + Salamander-обфускация. Маскируется под HTTP/3.
- **[[mKCP]]** — Xray transport (reliable-UDP), маскировка под BitTorrent/SRTP/WeChat header.
- **AmneziaWG** ([[AmneziaWG]]) — обфусцированный WireGuard (тоже UDP, но другая нiches: WG-stack).

В РФ-2026 — **partial**: UDP/443 как QUIC проходит, **если IP/AS в whitelist**; иначе блокируется на L3. На non-whitelist mobile — UDP throttle / drop.

## Сравнение

| Параметр | **[[Hysteria-2]]** | **[[mKCP]]** | **[[AmneziaWG]]** |
|---|---|---|---|
| База | QUIC (RFC 9000) | KCP (custom reliable-UDP) | WireGuard |
| Уровень | proxy-протокол | Xray transport | full VPN |
| Зависит от | свой stack | Xray/V2Ray | WG-kernel + Amnezia patches |
| TLS-маскировка | да (`h3` ALPN) | нет (header-маска) | нет (WG-style handshake) |
| Congestion | **Brutal** (декларативный) | KCP-FEC | WG-стандарт |
| Обфускация | Salamander | seed + header-type | junk-packets, magic-replace |
| Поведение на 5% loss | ~95% throughput | заметная просадка | средне |
| РФ-2026 status | **partial** | **partial** (с регрессиями) | **working** для UDP вне whitelist |
| Маскировка под | HTTP/3 (YouTube) | uTorrent/SRTP/WeChat | (никакую — голый UDP) |

## Какую проблему решает класс
- TCP-VPN страдает от **session-freezing** в РФ DPI (~16 КБ затем замороз без RST).
- QUIC поверх UDP **не имеет долгого state** — каждый пакет более независим.
- На **mobile с lossy radio** UDP-туннели быстрее TCP-VPN (custom congestion control).
- Часть QUIC-трафика — обычный (HTTP/3 на YouTube/Google) → DPI не может блокировать всё подряд.

## Где ломается весь класс
- **UDP-блокировка / throttling:** некоторые провайдеры лимитируют UDP-throughput (особенно non-443) → throttling.
- **Whitelist на UDP/443:** только реальные QUIC-серверы (Google, Cloudflare, Yandex) пропускаются.
- **Active probing:** атакующий шлёт фальшивый QUIC-init на твой IP — твой сервер отвечает иначе → детект. Salamander/seed-обёртки помогают.
- **NAT timeout** на UDP короткий (30 с обычно) → нужен keep-alive.

## Когда что использовать
- **Хочу простой VPN с лучшим throughput на lossy** → [[Hysteria-2]].
- **Уже поднят Xray, не хочу второй стек** → [[mKCP]] как transport под VLESS.
- **Mobile-сеть, UDP к моему IP проходит, но whitelist** → [[AmneziaWG]] (другой ниши).
- **Mobile в whitelist-режиме, UDP вообще дропается** → возвращаться к TCP-стеку: [[VLESS-Reality]] + [[xHTTP]].

## Связи
- **Базируется на:** [[QUIC]], [[UDP]].
- **Атомарные заметки:** [[Hysteria-2]], [[mKCP]].
- **Альтернативный класс (TCP):** [[VLESS-Reality]], [[Shadowsocks-2022]], [[xHTTP]].
- **UDP-альтернатива не-QUIC:** [[AmneziaWG]].

## Источники
- src-08 — обзор QUIC/mKCP/Hysteria среди транспортов 2024.
- src-06 — упоминание UDP-каскадов.
