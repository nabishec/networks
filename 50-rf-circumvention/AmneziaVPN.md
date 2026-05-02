---
title: AmneziaVPN
aliases: ["Amnezia", "AmneziaWG"]
type: tool
layer: application
status: working
status_as_of: 2026-05-02
risk: low
prerequisites: ["[[VPN]]", "[[Туннелирование]]"]
related: ["[[VLESS-Reality]]", "[[Xray-core]]", "[[IPsec]]", "[[AmneziaWG]]"]
sources: [src-02, src-06, src-07, src-08]
tags: [networking, rf-circumvention, tool]
---
# AmneziaVPN / AmneziaWG

## TL;DR
**Open-source self-hosted VPN-stack** с упрощённым deploy'ом для не-технических пользователей. Server-component поднимается на VPS «нажатием кнопки» через Amnezia-клиент → клиент сам генерирует конфиг и устанавливает. Поддерживает: **OpenVPN с обфускацией (Cloak)**, **WireGuard**, **AmneziaWG** (modified WG с anti-DPI features), **Shadowsocks**, **XRay-VLESS-Reality**.

## Что делает
- **Self-hosted:** свой сервер, свой trust.
- **Deploy через клиент:** Windows/macOS/Linux/Android/iOS-клиент сам ssh'ится в VPS, ставит docker-compose, генерирует ключи.
- **Multi-protocol:** на одном сервере — несколько протоколов; клиент переключается.
- **AmneziaWG:** WireGuard + случайные junk-pakets + изменённые header-magic-bytes для обхода WG-pattern-detection.

## AmneziaWG vs стандартный WireGuard
| Feature | WireGuard | AmneziaWG |
|---|---|---|
| Detection | DPI ловит handshake-pattern | случайные модификации header'а |
| Junk packets | нет | case-by-case |
| In-tunnel obfuscation | нет | есть |
| Совместимость | RFC | own-flavor (нужен Amnezia-клиент) |

## Установка (упрощённо)
1. Скачать Amnezia-client (https://amnezia.org).
2. Запустить → «New VPN» → ввести SSH-credentials VPS.
3. Выбрать протоколы (Cloak, WG, Reality).
4. Кнопка «Install» → клиент сам ставит сервер.
5. Получает конфиг, подключается.

## Где взять
- GitHub: https://github.com/amnezia-vpn
- Релизы: https://amnezia.org/

## Связи
- **Использует:** OpenVPN (с [[Cloak]]-плагином), WireGuard, [[Xray-core]] (для VLESS-Reality), [[Shadowsocks-2022]].
- **Соседи по уровню:** [[Hiddify]] (более универсальный), [[3X-UI и Marzban]] (advanced multi-user).
- **Подходит для:** не-технического self-hosted VPN.

## Источники
- src-02, src-06, src-07, src-08.
