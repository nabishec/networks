---
title: Marzban
aliases: ["Marzban panel", "Gozargah Marzban"]
type: tool
layer: application
status: working
status_as_of: 2026-05-02
risk: low
prerequisites: ["[[Xray-core]]", "[[HTTPS]]"]
related: ["[[3X-UI и Marzban]]", "[[3X-UI]]", "[[Remnawave]]", "[[Hiddify]]"]
sources: [src-06]
tags: [networking, rf-circumvention, tool]
---
# Marzban — multi-node web-панель для Xray

## TL;DR
**Marzban** (Gozargah/Marzban, Python+ReactJS) — **multi-node** web-панель для **Xray-core**. Один **master** управляет несколькими **node-серверами** через mTLS-канал; пользователь и его подписка едины — реальный VPN-инфраструктурный пакет. Поддерживает VMess (TCP/WS), VLESS (включая Reality), Trojan, Shadowsocks (классический, **не 2022**). Ставится только через **Docker**. Ниша: «провайдер для community», 5–50 нод, биллинг через Telegram-бот.

## Что делает
- **Master + Nodes:** master хранит users/subscriptions/stats; nodes — голые Xray, к которым идёт client-трафик. Регистрация ноды через `marzban-node` + сертификат.
- **Единая подписка** на пользователя → клиент получает список **всех нод** во всех геолокациях, переключается между ними при падении.
- **Quota / expire-date / IP-online-limit / re-auth** на пользователя.
- **Telegram-бот** для self-service (`marzban-telegram-bot` extension).
- **Hosts-конфигурация:** для одного inbound можно объявить несколько `hostname` (CDN-fronting, Self-Steal, прямой IP) — клиент попробует все.
- **API:** REST (`/api/users`, `/api/nodes`) + WebSocket → можно интегрировать со своим биллингом.

## Где взять / установка

```bash
sudo bash -c "$(curl -sL https://github.com/Gozargah/Marzban-scripts/raw/master/marzban.sh)" @ install
```
Скрипт ставит **Docker + docker-compose**, поднимает **master** в контейнерах, генерирует SSL-cert.

**После установки:**
1. Открыть `https://<server-ip>:8000/dashboard/`.
2. Создать admin (`marzban cli admin create --sudo`).
3. Добавить **inbound** (VLESS-Reality рекомендуется).
4. Создать первого user — получить subscription-link.

**Добавить ноду** (на отдельном VPS):
```bash
sudo bash -c "$(curl -sL https://github.com/Gozargah/Marzban-scripts/raw/master/marzban-node.sh)" @ install
```
- В master-UI: Nodes → Add Node → ввести IP + сертификат-pair.
- Master толкает Xray-конфиг на node через mTLS gRPC.

## Где ломается / ограничения
- **Только Docker:** для bare-metal без Docker — нельзя, что неудобно на маленьких VPS.
- **Shadowsocks — не 2022:** для современного [[Shadowsocks-2022]] придётся править вручную или брать [[Remnawave]].
- **Master = SPOF:** падение master → новые подписки/login недоступны (трафик через nodes продолжает идти).
- **DB на SQLite (default):** для 1000+ users — переходить на MariaDB/MySQL (поддерживается).
- **Subscription-link нужно прятать за CDN/Cloudflare:** иначе РКН блокирует домен → клиенты не получают новые конфиги.
- **Master-side log retention:** по умолчанию хранит трафик-метрики; для приватности — отключать или ротейтить.

## Сравнение: когда **именно** Marzban
- **3+ ноды в разных странах под одной админкой** → Marzban.
- **Self-service через Telegram-бот, без сложного биллинга** → Marzban достаточно.
- **1 нода, простая admin-панель** → [[3X-UI]] легче.
- **Нужен Subscribe-ID + dynamic chains + heavy automation** → [[Remnawave]].

## Связи
- **Базируется на:** [[Xray-core]] (backend на каждой ноде).
- **Конкуренты:** [[3X-UI]] (single-node), [[Remnawave]] (тяжелее, больше automation).
- **Клиенты:** [[Hiddify]], NekoBox, v2raytun, Streisand через subscription-URL.
- **Сводка:** [[3X-UI и Marzban]].

## Источники
- src-06 — Marzban в списке Xray-управляющих панелей для каскадных архитектур РФ-обхода.
- [Устанавливаем и настраиваем прокси-сервер Marzban — habr 771892](https://habr.com/ru/articles/771892/)
- [Marzban vs 3X-UI: разбираемся в сортах панелей — habr 982492](https://habr.com/ru/articles/982492/)
- [Gozargah/Marzban GitHub](https://github.com/Gozargah/Marzban)
