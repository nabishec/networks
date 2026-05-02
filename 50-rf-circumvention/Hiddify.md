---
title: Hiddify
aliases: ["Hiddify-Next", "Hiddify"]
type: tool
layer: application
status: working
status_as_of: 2026-05-02
risk: low
prerequisites: ["[[VPN]]"]
related: ["[[Sing-box]]", "[[Xray-core]]"]
sources: [src-03, src-06, src-08]
tags: [networking, rf-circumvention, tool]
---
# Hiddify (Hiddify-Next)

## TL;DR
**Cross-platform клиент** для обхода блокировок (Windows, macOS, Linux, Android, iOS). Использует **sing-box** как backend; UI полностью заточен под use-case РФ/Иран. Поддерживает импорт VLESS/Trojan/SS/Hysteria-link'ов, **subscribe-URL** (auto-update конфигов), TUN-режим (полный VPN), split routing с geo-файлами.

## Что делает
- Импорт ссылок (`vless://`, `trojan://`, `ss://`, `hysteria2://`).
- **Subscriptions:** URL-подписка → автоматическое получение списка серверов (Marzban, 3X-UI генерируют такие).
- **TUN-mode:** все системные приложения через VPN.
- **Split routing:** preset'ы для РФ/Ирана/Китая.
- **DNS-leak protection.**

## Где взять
- GitHub: https://github.com/hiddify/hiddify-next
- App stores: Google Play (часто блокируется), Apple App Store, F-Droid.

## Использование
1. Установить.
2. Получить **VLESS/Reality-link** от своего сервера или сервиса.
3. «+» → импорт ссылки.
4. Включить tunnel.
5. Опционально: подписать subscription (если используешь панель типа Marzban).

## Альтернативы
- **NekoBox / NekoboxForAndroid** — UI попроще.
- **v2rayN / v2rayNG** — Windows/Android.
- **Streisand, FoxRay, v2raytun, Shadowrocket** — iOS.
- **Happ** — современный iOS-клиент.

## Связи
- **Использует:** [[Sing-box]] (backend).
- **Поддерживает:** [[VLESS-Reality]], [[xHTTP]], [[Shadowsocks-2022]], [[QUIC и mKCP]] (Hysteria), [[Split routing]].
- **Управляется через:** Marzban-subscriptions, 3X-UI link'и.

## Источники
- src-03, src-06, src-08.
