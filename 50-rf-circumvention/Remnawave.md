---
title: Remnawave
aliases: ["Remnawave Panel", "Remnawave"]
type: tool
layer: application
status: working
status_as_of: 2026-05-02
risk: low
prerequisites: ["[[Xray-core]]", "[[HTTPS]]"]
related: ["[[3X-UI и Marzban]]", "[[3X-UI]]", "[[Marzban]]", "[[Hiddify]]"]
sources: [src-02, src-06]
tags: [networking, rf-circumvention, tool]
---
# Remnawave — heavy-automation web-панель для Xray

## TL;DR
**Remnawave** (Node.js + React, GitHub `Remnawave/Panel`) — **современная** «тяжёлая» альтернатива [[Marzban]]. Заточена под **профессиональных VPN-провайдеров**: десятки нод, **Subscribe-ID** (динамическая ротация chain'ов и inbound'ов **без переподписки клиента**), **Python-SDK** для интеграции с Telegram-ботами и биллингом. Ставится через Docker; жор RAM **≥ 2 ГБ** только под панель. По заявлениям: одна нода под Remnawave-управлением держит ~10 000 active sessions.

## Что делает
- **Multi-node**, как [[Marzban]], но с более продвинутой логикой:
  - **Subscribe-ID:** один URL подписки = stable identity клиента; админ меняет inbound'ы/нагрузку, клиент ничего не замечает.
  - **Inbound-pool с ротацией:** автоматическое распределение users по нодам по latency / load.
- **Node.js Backend + React Frontend:** UI динамичнее, чем у Marzban (real-time-метрики через WebSocket).
- **Python SDK:** `remnawave-sdk` для Telegram-ботов, биллинг-интеграций, кастомных provisioning-скриптов.
- **Migration tools:** официальный скрипт миграции **Marzban → Remnawave** (`vff-remnawave-auto`), сохраняет user-base.
- **Reverse-proxy ready:** часто разворачивают **за** Caddy/Nginx с Cloudflare-fronting (см. `remnawave-reverse-proxy`).
- **Поддерживает Shadowsocks-2022** (в отличие от Marzban).

## Где взять / установка

**GitHub:** https://github.com/Remnawave/Panel.
**Docs:** https://docs.rw/.

**Базовый деплой (Docker):**
```bash
git clone https://github.com/Remnawave/Panel.git
cd Panel
cp .env.sample .env
# Редактируем JWT_SECRET, домен, БД (PostgreSQL)
docker compose up -d
```

**За reverse-proxy (рекомендуется):**
- Caddy с автоматическим Let's Encrypt → проксирует `panel.example.com` → `localhost:3000`.
- Cloudflare перед Caddy → скрывает реальный IP master-сервера.

**Добавление ноды:** `remnawave-node` Docker-образ + master-cert; регистрация в UI.

## Где ломается / ограничения
- **RAM-голод:** минимум 2 ГБ только под master; для 10 нод и ~5к users — 4–8 ГБ.
- **PostgreSQL обязателен** (в отличие от SQLite-варианта Marzban).
- **Сложнее в первый раз:** больше движущихся частей (panel, subscription-page, sub-tg-bot, node-agent).
- **Документация на английском/русском, но API меняется быстро** — pin'ить версии в production.
- **DNS-leak в Xray-конфиге** — в community Q&A был кейс утечек DNS при дефолтной Remnawave-генерации; проверять `outbounds[].streamSettings.sockopt.domainStrategy=UseIP`.

## Сравнение: когда **именно** Remnawave
- **«Серьёзный VPN-провайдер»** на 1 000+ users, биллинг, Telegram-бот, dynamic chains → Remnawave.
- **Миграция с Marzban** на более новую панель (multi-node + Subscribe-ID) → Remnawave.
- **3–5 нод, простая админка** → [[Marzban]] хватит.
- **1 нода, личный/team-уровень** → [[3X-UI]].

## Связи
- **Базируется на:** [[Xray-core]] (backend).
- **Конкуренты:** [[Marzban]] (Python, легче), [[3X-UI]] (single-node).
- **Клиенты:** [[Hiddify]], v2raytun, NekoBox через subscription-URL.
- **Совместима с:** [[Shadowsocks-2022]], [[VLESS-Reality]], [[xHTTP]], [[Hysteria-2]] (как inbound на ноде).
- **Сводка:** [[3X-UI и Marzban]].

## Источники
- src-02 — Remnawave в списке инструментов цепочки.
- src-06 — упомянута среди современных панелей.
- [Marzban vs 3X-UI — habr 982492](https://habr.com/ru/articles/982492/) (контекст для сравнения).
- [vff-remnawave-auto — Marzban → Remnawave migration tool](https://github.com/ryabkov82/vff-remnawave-auto)
- [remnawave-reverse-proxy](https://github.com/eGamesAPI/remnawave-reverse-proxy)
- [Remnawave/Panel GitHub](https://github.com/Remnawave/Panel)
