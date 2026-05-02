---
title: 3X-UI / Marzban / Remnawave — сравнение веб-панелей Xray
aliases: ["VPN-панели", "3X-UI и Marzban", "панели Xray"]
type: tool
layer: application
status: working
status_as_of: 2026-05-02
risk: low
prerequisites: ["[[Xray-core]]", "[[HTTPS]]"]
related: ["[[3X-UI]]", "[[Marzban]]", "[[Remnawave]]", "[[Hiddify]]"]
sources: [src-03, src-06, src-02]
tags: [networking, rf-circumvention, tool]
---
# 3X-UI / Marzban / Remnawave — сравнение

> **Это сводная заметка.** Атомные детали — в [[3X-UI]], [[Marzban]], [[Remnawave]].

## TL;DR
Три популярные **web-панели управления Xray-сервером**, заменяющие правку JSON-конфигов руками. Все основаны на [[Xray-core]]. Различаются масштабом и сложностью:
- **[[3X-UI]]** — простая, single-node, личное/team-использование.
- **[[Marzban]]** — middle-weight, multi-node через master+workers.
- **[[Remnawave]]** — heavy, для «провайдеров»: Subscribe-ID, dynamic chains, Python-SDK.

## Сравнение

| Параметр | **[[3X-UI]]** | **[[Marzban]]** | **[[Remnawave]]** |
|---|---|---|---|
| Stack | Go + Vue | Python + ReactJS | Node.js + React |
| Установка | bash one-liner / Docker | Docker only | Docker + PostgreSQL |
| Multi-node | нет (костылями) | **да** (master + nodes mTLS) | **да** (с Subscribe-ID) |
| RAM на master | ~100 МБ | ~300 МБ | **≥ 2 ГБ** |
| Shadowsocks-2022 | да | **нет** (только классический) | да |
| Telegram-бот | плагин | extension | SDK (Python) |
| API | REST | REST + WS | REST + WS + Python SDK |
| Subscribe-ID | нет | нет | **да** |
| Dynamic chain rotation | нет | частично | **да** |
| Целевая аудитория | личное / 1–100 users | community / 100–1000 | provider / 1000+ |
| Сложность first-setup | низкая | средняя | высокая |

## Что они делают (общее)
- **Конфигурация inbound'ов:** VLESS, VMess, Trojan, Shadowsocks, Hysteria-2, mKCP, xHTTP без правки JSON.
- **User management:** UUID + quota + expire-date + IP-online-limit.
- **Subscription URL:** клиенту ([[Hiddify]], NekoBox, v2raytun) — единый URL, обновляющий список серверов.
- **Traffic accounting / live-метрики.**
- **Maska-helper'ы:** `genUUID`, `x25519` для Reality, ACME для Let's Encrypt.

## Где взять
- 3X-UI: https://github.com/MHSanaei/3x-ui — см. [[3X-UI]].
- Marzban: https://github.com/Gozargah/Marzban — см. [[Marzban]].
- Remnawave: https://github.com/Remnawave/Panel — см. [[Remnawave]].

## Алгоритм выбора
1. **1 VPS, до ~100 пользователей, простой setup за 10 минут** → [[3X-UI]].
2. **3+ VPS в разных странах под одной админкой, единая подписка** → [[Marzban]].
3. **Биллинг, Telegram-бот, dynamic chains, миграция > 1000 users** → [[Remnawave]].
4. **Не хочу панель вообще** → [[Xray-core]] + редактируешь JSON руками + systemd.

## Связи
- **Базируется на:** [[Xray-core]] (backend) — все три панели генерируют именно Xray-config.
- **Атомарные заметки:** [[3X-UI]], [[Marzban]], [[Remnawave]].
- **Соседи по уровню:** Sing-box-based panels (Hiddify-Manager).
- **Используется с клиентами:** [[Hiddify]], NekoBox, v2raytun — через subscription-URL.

## Источники
- src-03 — 3X-UI явно как часть 4-уровневой архитектуры.
- src-06 — Marzban, Remnawave, 3x-ui упомянуты в связке инструментов.
- src-02 — Remnawave среди инструментов цепочки.
- [Marzban vs 3X-UI: разбираемся в сортах панелей — habr 982492](https://habr.com/ru/articles/982492/)
