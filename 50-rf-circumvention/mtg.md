---
title: mtg
aliases: ["mtg", "9seconds/mtg", "mtg MTProxy"]
type: tool
layer: application
status: working
status_as_of: 2026-05-02
risk: low
prerequisites: ["[[MTProxy и FakeTLS]]", "[[TLS — рукопожатие]]", "[[HMAC]]"]
related: ["[[PB8 — MTProxy + FakeTLS]]", "[[PingZen]]"]
sources: [src-10]
tags: [networking, rf-circumvention, tool]
---
# mtg — Go-реализация MTProxy

## TL;DR
**`9seconds/mtg`** — самая популярная **Go-реализация** Telegram-MTProxy на 2026 г.; де-факто заменила официальный C-репозиторий Telegram, потому что у официального **нет** FakeTLS, а у `mtg` он встроен. Один статический бинарник + `simple-run`-команда + автогенерация secret'а с domain'ом маскировки. Используется в [[PB8 — MTProxy + FakeTLS]] как сервер.

## Что делает
- **Server-side MTProxy:** принимает Telegram-клиентов, расшифровывает MTProto-handshake, форвардит к Telegram-DC.
- **FakeTLS-режим (EE-secret):** маскирует первые байты под TLS 1.3 ClientHello к указанному домену (`www.google.com`, `cdn.discordapp.com` и т.д.).
- **Anti-replay** + HMAC-SHA256 hidden-handshake (без знания secret'а ClientHello валидно как обычный TLS, но не открывает proxy-режим).
- **Multi-secret + per-user-stats** (для публичных прокси).
- **`access` команда** — генерирует `tg://proxy?...`-link и QR.

## Установка (Linux)
```bash
# Скачать релиз с GitHub releases:
wget https://github.com/9seconds/mtg/releases/download/v2.x.x/mtg-2.x.x-linux-amd64.tar.gz
tar -xf mtg-*.tar.gz && sudo mv mtg /usr/local/bin/

# Сгенерировать FakeTLS-secret:
mtg generate-secret tls --hex www.google.com
# Output: ee<random-32-hex>7777772e676f6f676c652e636f6d
#         ↑ префикс "ee" = FakeTLS, хвост = hex(domain)
```

## Запуск
```bash
# Простой запуск:
mtg simple-run 0.0.0.0:443 SECRET --concurrency 8192

# systemd-unit (рекомендуется):
sudo tee /etc/systemd/system/mtg.service <<EOF
[Unit]
Description=mtg MTProxy
After=network.target

[Service]
ExecStart=/usr/local/bin/mtg simple-run 0.0.0.0:443 SECRET
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF
systemctl enable --now mtg
```

## Получить tg://link
```bash
mtg access SECRET --ip-v4 SERVER-IP
# tg://proxy?server=SERVER-IP&port=443&secret=ee...
```
Вставить в Telegram → Settings → Data and Storage → Proxy → Add → MTProxy.

## Почему Go-реализация, а не оф. C-MTProxy
- Оф. репо Telegram (`TelegramMessenger/MTProxy`) — **без FakeTLS**, обнаружим простым DPI на post-handshake-pattern (src-10).
- Сборка C-MTProxy — глюки с openssl-версиями, нужен `make` + патчи.
- Go: один static-binary, без зависимостей, кросс-платформа.
- **`telemt`** — Rust-альтернатива; функционально близка, реже обновляется.

## Где ломается
- **Post-handshake DPI** (src-10): ТСПУ ловит MTProxy через behavioral pattern даже с FakeTLS — срок жизни IP **дни-недели**.
- **«Золотые прокси»** (SNI = `yandex.ru`/`gosuslugi.ru`/`sberbank.ru`) дольше живут — ТСПУ боится сломать сами эти домены, — но юридически серая зона.
- **PingZen и аналоги** (см. [[PingZen]]) активно мониторят публичные `mtg`-инстансы → blacklist популярных IP за часы.
- **Оф. сертификат не нужен**, но если включён `--secure-only` — secret должен быть EE-формата.

## Связи
- **Реализует:** [[MTProxy и FakeTLS]] (FakeTLS+Obfuscated2).
- **Используется в:** [[PB8 — MTProxy + FakeTLS]].
- **Соседи по уровню:** `telemt` (Rust), официальный `MTProxy` (C, deprecated для FakeTLS).
- **Проверяется через:** [[PingZen]] (health-check); [[hyperion-cs DPI-checker]] (общая DPI-диагностика).

## Где взять
- GitHub: https://github.com/9seconds/mtg
- Releases: https://github.com/9seconds/mtg/releases

## Источники
- src-10 (MTProxy / mtg / telemt в РФ-2026).
- Habr: [Повышаем стабильность Telegram: поднимаем партизанский MTProxy с Fake TLS](https://habr.com/ru/articles/994934/) — практический setup `9seconds/mtg:2` через Docker.
- Habr: [Настраиваем MTProto прокси с Fake TLS за 5 минут](https://habr.com/ru/articles/1010942/) — quickstart с `mtg generate-secret tls`.
