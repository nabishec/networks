---
title: CDN-фронтинг
aliases: ["Domain fronting", "CDN fronting", "Cloudflare-фронтинг"]
type: technique
layer: application
status: partial
status_as_of: 2026-05-02
risk: medium
prerequisites: ["[[CDN — устройство]]", "[[TLS — рукопожатие]]", "[[HTTPS]]"]
related: ["[[VLESS-Reality]]", "[[Yandex API Gateway фронтинг]]"]
sources: [src-03, src-08]
tags: [networking, rf-circumvention, technique]
---
# CDN-фронтинг

## TL;DR
Прокси-сервер прячется **за CDN** (Cloudflare, Amazon CloudFront, Yandex CDN). Клиент шлёт TLS-handshake с SNI популярного сайта (например, `*.cloudflare.com`), но в HTTP-headers внутри указывает свой backend → CDN маршрутизирует. DPI видит «легитимное соединение к Cloudflare». Классический **domain fronting** запрещён большинством крупных CDN с 2018 г., но варианты с **own-domain поверх Cloudflare** (без cross-domain трюков) ещё работают.

## Какую проблему решает
Спрятать backend за репутацию CDN — DPI вряд ли заблокирует Cloudflare целиком (сломал бы пол-интернета). Поэтому фронт-IP CDN остаётся доступным даже когда сам backend в чёрном списке.

## Как работает

**Domain fronting (классический, 2014-2018):**
- TLS SNI = популярный домен на CDN (`outer.example.com`).
- HTTP `Host:` header = свой бэкенд (`hidden.example.com`).
- CDN видит SNI → принимает; смотрит Host → маршрутизирует на свой backend.
- DPI снаружи видит только SNI → outer.

**Заблокировано** Cloudflare (2018), Google (2018), Amazon (2018) — теперь **SNI должен совпадать с Host**.

**Modern fronting через own-domain:**
- Свой домен `proxy.mysite.com` через Cloudflare proxy.
- TLS-handshake к Cloudflare-IP с SNI=`proxy.mysite.com`.
- Cloudflare прозрачно проксирует в backend.
- DPI видит «соединение к Cloudflare-IP» — всё равно валидно, если backend-IP скрыт.

**Cloudflare Workers / WARP:**
- Workers: serverless-логика на edge — можно проксировать.
- WARP: VPN-сервис самой Cloudflare (туннель внутри их сети).

## Где ломается / почему может не работать
- **Чистый domain fronting сломан** на крупных CDN.
- В **РФ-2025+** (src-09): РКН блокирует **целые AS** (включая Cloudflare 13335) или применяет **whitelist** доменов на L7 → SNI=cloudflare.com может не пропускаться, если домен не в whitelist.
- **Cloudflare-tunnel (Argo)** более устойчив, но требует свой домен.
- Бесплатные планы Cloudflare имеют **rate-limit** и ограничение пропускной способности.

## Минимальный пошаговый сценарий
1. Регистрация домена (`example.com`).
2. Перевод DNS в Cloudflare.
3. CNAME `proxy.example.com` → backend-IP (с включённым «proxy» — оранжевая туча).
4. На backend — Xray с TLS-сертификатом `proxy.example.com`.
5. Клиент: VLESS-link с `address=proxy.example.com`, `sni=proxy.example.com`.
6. Трафик идёт: клиент → Cloudflare-edge → backend.

## Что нужно
- Свой домен.
- Cloudflare-аккаунт (free).
- Backend-сервер (любой VPS вне РФ).
- Xray + TLS-сертификат (можно от Cloudflare-CA).

## Связи
- **Базируется на:** [[CDN — устройство]] (механизм edge-proxy), [[HTTPS]] (TLS), [[TLS — рукопожатие]].
- **Используется в:** [[PB3 — 4-уровневая архитектура за 265₽]] (один из уровней).
- **Соседи по уровню:** [[Yandex API Gateway фронтинг]] (РФ-аналог), [[VLESS-Reality]] (другая школа маскировки).
- **Противопоставляется:** Reality — Reality не нужен CDN; CDN-фронтинг даёт другой профиль маскировки.

## Источники
- src-03, src-08.
