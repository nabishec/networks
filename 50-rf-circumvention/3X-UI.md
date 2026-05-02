---
title: 3X-UI
aliases: ["3x-ui", "X-UI", "MHSanaei 3x-ui"]
type: tool
layer: application
status: working
status_as_of: 2026-05-02
risk: low
prerequisites: ["[[Xray-core]]", "[[HTTPS]]"]
related: ["[[3X-UI и Marzban]]", "[[Marzban]]", "[[Remnawave]]", "[[Hiddify]]"]
sources: [src-03, src-06]
tags: [networking, rf-circumvention, tool]
---
# 3X-UI — простая web-панель для Xray

## TL;DR
**3X-UI** (форк X-UI от MHSanaei) — самая популярная **single-node** web-панель для **Xray-core**. GUI на Go+Vue, ставится одним bash-скриптом, по умолчанию слушает на `:2053`. Поддерживает **VMess, VLESS (включая Reality), Trojan, Shadowsocks, WireGuard, Hysteria, mKCP, xHTTP, gRPC**. Ниша: **personal / small-team** (до сотен пользователей на одной ноде). Multi-node — **только костылями**, для master-slave идти к [[Marzban]] / [[Remnawave]].

## Что делает
- **Inbound configurator:** через UI собирается JSON-конфиг Xray (без правки руками).
- **Per-user UUID + traffic-limits + expiration date.**
- **Subscription URL:** `https://panel/sub/<token>` — клиенты ([[Hiddify]], NekoBox, v2rayN) импортируют как подписку.
- **Live-метрики:** трафик, online-clients, CPU/RAM сервера, IP-online-limit.
- **Telegram-бот** для self-service (выдача ссылок, top-up).
- **Backup/restore** конфигурации через UI.
- **Реality-helper:** на странице inbound'а есть кнопки «сгенерировать UUID / x25519 / shortid».

## Где взять / установка

```bash
bash <(curl -Ls https://raw.githubusercontent.com/MHSanaei/3x-ui/master/install.sh)
```
Скрипт спросит **admin / password / port**. По умолчанию — `2053`.

**Открыть:** `https://<server-ip>:<port>/panel/`. Логин `admin/admin` если ничего не меняли.

**Docker-альтернатива:**
```bash
docker run -d --network host -v $PWD/db:/etc/x-ui ghcr.io/mhsanaei/3x-ui:latest
```

**После установки в РФ-2026 первым делом:**
1. Сменить admin-port на нестандартный (`30000+`).
2. Закрыть UI за reverse-proxy с basic-auth или Cloudflare-Access.
3. Включить **fail2ban** на UI-port.
4. Настроить inbound: VLESS-Reality на 443 (см. [[VLESS-Reality]], [[PB7 — basic VLESS-Reality с нуля]]).

## Где ломается / ограничения
- **Single-node by design:** один master = один Xray-сервер. Для географически распределённой сети — нужен [[Marzban]] (multi-node) или [[Remnawave]].
- **UI с публичного IP — мишень:** защищать обязательно (reverse-proxy, geo-fence, fail2ban). Иначе — bruteforce.
- **Subscription-link нужен HTTPS:** иначе клиенты не примут (Apple/iOS особенно строги).
- **Утечки в логах:** при дефолтных настройках access-log пишет UUID — чистить.

## Сравнение: когда **именно** 3X-UI
- **1 VPS, до ~100 пользователей** → 3X-UI.
- **Несколько VPS в разных странах, единая админка** → [[Marzban]].
- **«Серьёзный провайдер»**, telegram-боты, биллинг, Subscribe-ID → [[Remnawave]].

## Связи
- **Базируется на:** [[Xray-core]] (backend).
- **Конкуренты:** [[Marzban]] (multi-node), [[Remnawave]] (heavy automation).
- **Клиенты:** [[Hiddify]], NekoBox, v2rayN, v2raytun — через subscription-URL.
- **Сводка:** [[3X-UI и Marzban]].

## Источники
- src-03 — 4-уровневая архитектура за 265₽: 3X-UI как главная панель.
- src-06 — упомянут в наборе Xray-инструментов.
- [3X-UI: Shadowsocks-2022 & XRay (XTLS) сервер с простой настройкой — habr 735536](https://habr.com/ru/articles/735536/)
- [Marzban vs 3X-UI: разбираемся в сортах панелей — habr 982492](https://habr.com/ru/articles/982492/)
- [GitHub: MHSanaei/3x-ui](https://github.com/MHSanaei/3x-ui)
