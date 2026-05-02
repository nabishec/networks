---
title: AS-level whitelist
aliases: ["ASN whitelist", "Блокировка по AS"]
type: concept
layer: network
status: working
status_as_of: 2026-05-02
risk: low
prerequisites: ["[[Автономная система]]", "[[BGP]]"]
related: ["[[Белые списки]]", "[[ТСПУ]]"]
sources: [src-09]
tags: [networking, rf-circumvention, concept]
---
# AS-level whitelist

## TL;DR
Стратегия РФ-DPI: фильтровать **на уровне AS** (Autonomous System), не отдельных IP. **Один AS = миллионы IP** одним правилом. Эффективно для админa, но **коллатеральный ущерб огромен**: src-09 (16.02.2025, повторно сканировано 10.02.2026) — whitelist для 72 AS, **затронул 391 AS** (~225 млн IP), **много легитимных сервисов** упало вместе с целью. AS Cloudflare (13335), OVH (16276), Hetzner (24940), DigitalOcean (14061) попали под раздачу.

## Какую проблему решает (с точки зрения регулятора)
- Простота администрирования: один правило на ASN покрывает все его IP.
- IP-Whitelist на 72 AS = ~few-hundred-million IP «легитимных».
- Адаптивность: новые IP внутри уже-whitelisted AS не требуют re-config.

## Как работает

**Источник AS:**
- IRR (Internet Routing Registry) — RIPE/ARIN/APNIC.
- BGP-feed: PeeringDB, RouteViews.
- Внутренние списки регулятора.

**Реализация на ТСПУ:**
- Lookup-table: для каждого пакета `dst_IP → AS` (через BGP-table).
- Если AS в whitelist → пропустить; иначе → drop.

**Что **может** быть в whitelist:**
- AS Yandex (8359, 13238).
- AS VK / Mail.ru (47764).
- AS Сбербанка, Госуслуг, ТАСС, государственных ресурсов.
- AS «трасты» (Lifecell, Beeline для роуминга).

**Не в whitelist (по src-09):**
- Cloudflare (13335).
- AWS (16509).
- OVH (16276).
- Hetzner (24940).
- DigitalOcean (14061).
- Linode (63949).

## Где ломается / почему может не работать (с точки зрения обходчика)
- **Найти whitelist-AS** и поднять там VPS.
- **Yandex Cloud** (AS 13238) — в whitelist, можно снять preemptible-VM или Cloud Functions.
- **VK Cloud** аналогично.
- **DiNet** (российский cloud) — иногда в whitelist.

**Подводный камень:** даже если AS в whitelist, конкретный домен/IP может быть в L7-blacklist → нужно проверять caso-by-caso.

## Минимальный пошаговый сценарий (диагностика)
1. **Обнаружить blacklist:** на мобильном устройстве с whitelist'ом → `curl https://example.com -m 5`. Если timeout/refused — ASN заблокирован.
2. **Проверить AS:** `whois example.com` или через https://bgp.he.net/ipinfo.
3. **Крос-проверить против whitelist** (cheburcheck.ru, src-09's blacklist).
4. **Если нужный сервис в blacklist-AS** — мигрировать на provider в whitelist-AS.

## Что нужно
- Знание BGP/ASN-системы.
- Доступ к публичным AS-data (PeeringDB, BGP looking glass).
- Опционально: cheburcheck.ru или dpi-detector.

## Связи
- **Базируется на:** [[Автономная система]] (концепт), [[BGP]] (как AS определяется), [[Иерархическая маршрутизация]].
- **Используется в:** [[Белые списки]] (главная страт.), [[ТСПУ]] (механизм).
- **Связано с:** [[Yandex API Gateway фронтинг]], [[CDN-фронтинг]] (обходные техники).

## Источники
- src-09 (детальный отчёт по 72/391 AS, 225 млн IP).
