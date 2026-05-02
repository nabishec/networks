---
title: "PB8 — MTProxy + FakeTLS"
aliases: ["PB8"]
type: playbook
layer: application
status: partial
status_as_of: 2026-05-02
risk: medium
prerequisites: ["[[MTProxy и FakeTLS]]", "[[TLS — рукопожатие]]", "[[HMAC]]"]
related: ["[[Хеш-функции]]"]
sources: [src-10]
tags: [networking, rf-circumvention, playbook]
---
# PB8 — MTProxy с FakeTLS-маскировкой (для Telegram)

## TL;DR
Для **Telegram-only** обхода (не для общего VPN): MTProxy с FakeTLS-режимом маскируется под TLS-handshake к указанному домену (например, `www.google.com`). Telegram-клиент использует MTProxy напрямую, **без** OS-VPN.

## Архитектура
```mermaid
flowchart LR
  TG[Telegram client] -->|MTProxy + FakeTLS| Server[VPS с mtg]
  Server -->|MTProto| Telegram-DC[Telegram Data Centers]
```

## Шаги

### 1. VPS вне РФ
- Linux (Ubuntu 22.04+).
- Open port 443 (или другой).
- Public IP.

### 2. Установить mtg (Go-реализация)
```bash
wget https://github.com/9seconds/mtg/releases/download/v2.x/mtg-2.x.x-linux-amd64.tar.gz
tar -xf mtg-*.tar.gz
sudo mv mtg /usr/local/bin/
```

### 3. Сгенерировать secret
```bash
mtg generate-secret tls --hex www.google.com
# Output: ee<...>7777772e676f6f676c652e636f6d
```
- `ee` префикс = FakeTLS-режим.
- Хвост — hex-кодированный domain для маскировки.

### 4. Запустить mtg
```bash
mtg simple-run 0.0.0.0:443 SECRET --concurrency 8192
# Или через systemd:
sudo tee /etc/systemd/system/mtg.service <<EOF
[Unit]
Description=mtg MTProxy
After=network.target

[Service]
ExecStart=/usr/local/bin/mtg simple-run 0.0.0.0:443 SECRET
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl enable --now mtg
```

### 5. Получить tg://-link
```bash
mtg access SECRET --ip-v4 SERVER-IP
# Output: tg://proxy?server=...&port=...&secret=...
```

### 6. Импорт в Telegram
- **Settings → Data and Storage → Proxy → Add Proxy → MTProxy**.
- Скопировать server, port, secret.
- Включить.

## Проверка
- В Telegram: значок настройки в верхнем углу → должен показывать «Connected via MTProxy».
- Tap на «Connection» → details. Если работает — Telegram открывается, channels работают.

## Где ломается
- **Post-handshake DPI detection** (src-10): ТСПУ может детектировать MTProxy через behavioral pattern даже с FakeTLS.
- **«Золотые прокси»** (маскировка под домены Yandex/Госуслуги/СберБанк) дольше держатся, но юридически сомнительные.
- **PingZen и аналоги** мониторят health публичных MTProxy → попадают в blacklist.
- Только **Telegram**: для общего VPN не подходит.

## Связи
- **Технический фундамент:** [[MTProxy и FakeTLS]].
- **Альтернативы:** [[PB7 — basic VLESS-Reality с нуля]] (общий VPN, работает и для Telegram).

## Источники
- src-10.
