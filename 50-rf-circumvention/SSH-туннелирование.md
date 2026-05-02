---
title: SSH-туннелирование
aliases: ["SSH tunnel", "SSH SOCKS"]
type: technique
layer: application
status: partial
status_as_of: 2026-05-02
risk: low
prerequisites: ["[[TCP]]", "[[Симметричная vs асимметричная криптография]]"]
related: ["[[VLESS-Reality]]"]
sources: [src-08]
tags: [networking, rf-circumvention, technique]
---
# SSH-туннелирование

## TL;DR
**Простейший VPN-альтернатив:** `ssh -D 1080 user@server` поднимает локальный SOCKS5-прокси, через который браузер тунелирует HTTP-трафик. SSH = TCP/22 (или нестандартный port), TLS-like-handshake (но не TLS). DPI обычно **не блокирует SSH** массово — это сломало бы DevOps. В РФ-2026 это **partial** работает: не для bulk, но для admin-tasks через VPS — ок.

## Какую проблему решает
- Минимум setup: только VPS с SSH и клиент (любая ОС).
- Шифрование на уровне SSH (ChaCha20-Poly1305 или AES-256-GCM).
- DPI редко блокирует SSH, потому что DevOps его использует массово.

## Как работает

**Динамическое перенаправление портов:**
```bash
ssh -D 1080 -N -f user@my-vps.com
# -D 1080 — локальный SOCKS5-сервер
# -N — без shell
# -f — в background
```

Клиент (браузер с FoxyProxy / system-wide) использует `socks5://127.0.0.1:1080`.

**Локальное перенаправление:**
```bash
ssh -L 8080:internal-server.com:80 user@vps.com
```
Локальный 8080 → tunnel → internal-server.

## Где ломается / почему может не работать
- **DPI-classifier для SSH:** РФ-DPI могут ограничивать SSH-сессии после X KB или замораживать длинные.
- **Whitelist-режим (мобильный):** SSH-IP не в whitelist → блокируется.
- **Performance:** SSH над TCP — не оптимизирован для bulk; one-stream для всего.
- **Лимиты bandwidth** на VPS-провайдере.

## Минимальный пошаговый сценарий
1. Купить VPS вне РФ (Хетцнер, OVH).
2. Создать SSH-ключ: `ssh-keygen -t ed25519`.
3. Скопировать на сервер: `ssh-copy-id user@vps.com`.
4. Запустить tunnel: `ssh -D 1080 -N user@vps.com`.
5. Браузер: SOCKS5 = 127.0.0.1:1080.

**Чтобы сделать persistence:** `autossh` или systemd-юнит с restart.

## Что нужно
- VPS вне РФ.
- SSH-клиент (везде есть).
- Опционально: `autossh` для reliable reconnect.
- Браузер с поддержкой SOCKS5 (FoxyProxy, OS-wide).

## Связи
- **Базируется на:** [[TCP]] (transport), [[Симметричная vs асимметричная криптография]] (SSH-keys).
- **Соседи по уровню:** [[VLESS-Reality]] (более масштабируемый), [[Shadowsocks-2022]] (другой simple proxy).
- **Противопоставляется:** Reality — для serious VPN; SSH — для quick-and-dirty.

## Источники
- src-08.
