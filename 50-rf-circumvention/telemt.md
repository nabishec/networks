---
title: telemt
aliases: ["telemt", "telemt-rs"]
type: tool
layer: application
status: working
status_as_of: 2026-05-02
risk: medium
prerequisites: ["[[MTProxy и FakeTLS]]", "[[TLS — рукопожатие]]"]
related: ["[[mtg]]", "[[VLESS-Reality]]", "[[Self-Steal — свой домен]]"]
sources: [src-10]
tags: [networking, rf-circumvention, tool]
---
# telemt

## TL;DR
**Rust + Tokio**-реализация MTProxy с FakeTLS. Ключевая «фича» — **transparent TCP splice**: если клиент пришёл без правильного HMAC-secret, telemt **не закрывает** соединение, а **прозрачно проксирует** его на реальный сайт (с настоящим TLS-сертификатом). Снаружи неотличим от обычного хостинга.

Размер: ~50 тыс. строк Rust (для сравнения: `mtg` Go ~25k, официальный C-MTProxy ~25k, эрланговская — ~3.5k). Telemt — самая «толстая» реализация, но и самая устойчивая к active probing.

## Какую проблему решает
- **Active probing** ([[Active probing]]): ТСПУ периодически пробует, что отвечает «странный TLS-сервер» в whitelist-AS. Если подключение не отдаёт никаких страниц или сразу разрывается — флажок «возможно, MTProxy».
- `mtg` (Go) при отсутствии secret разрывает TLS-handshake → детектируемый pattern.
- **telemt** при отсутствии secret продолжает TLS и **проксирует** запрос в указанный backend (например, реальный сайт), отдавая полноценный HTML с настоящим сертификатом. С точки зрения probe — это просто HTTPS-сервер.

## Как работает

### Splice-flow при probing
```
DPI/probe → telemt :443
              │
              ├─ ClientHello + HMAC-secret? → MTProxy-режим (TLS-record-обёртка для MTProto)
              │
              └─ ClientHello без HMAC → forward TCP-stream к backend
                                        backend: 127.0.0.1:8443 (nginx + реальный сайт)
                                        ↓
                                        Probe видит обычный HTTPS-сайт
```

В отличие от `mtg --multistream`, telemt делает splice **на уровне TLS-handshake**: HMAC-проверка идёт по байтам ClientHello.random; если не сошлось — соединение полностью передаётся в backend, без прерывания TLS.

### Конфиг
```toml
# telemt.toml
listen = "0.0.0.0:443"
secret = "ee<HEX>www.google.com"  # FakeTLS-secret + masquerade-domain
fallback = "127.0.0.1:8080"        # backend для не-MTProxy-клиентов
```

### Известные проблемы
- В РФ-сетях фиксировались **handshake timeouts** (см. issues и обсуждение в Habr-комментариях): часть РФ-провайдеров «затягивает» TCP-handshake к незнакомым IP — telemt при этом теряет свежие соединения.
- Большой бинарник (~15 MB), TLS-stack тянет за собой много transitive-крейтов.

## Минимальный сценарий
```bash
# Установка из релиза:
wget https://github.com/forhsd/telemt/releases/latest/download/telemt-linux-x86_64
chmod +x telemt-linux-x86_64

# Сгенерировать FakeTLS-secret:
./telemt-linux-x86_64 generate-secret tls --hex www.google.com
# → ee<32hex>776f777e676f6f676c652e636f6d

# Backend (реальный сайт) на :8080:
docker run -d -p 127.0.0.1:8080:80 nginx:alpine

# Запуск telemt:
./telemt-linux-x86_64 run --config telemt.toml
```

В Telegram: Settings → Proxy → MTProxy → server / port=443 / secret=ee...

## Связи
- **Альтернативы:** [[mtg]] (Go), официальный MTProxy от Telegram (C), 9seconds/mtg.
- **Концептуально похоже на:** [[Self-Steal — свой домен]] (forward к реальному backend при probe), [[VLESS-Reality]] (тот же приём для VLESS).
- **Используется с:** [[PingZen]] (мониторинг живости).
- **Источники документирующие:** [[MTProxy и FakeTLS]], [[Active probing]].

## Источники / Дальше читать
- [src-10 — Не соглашаясь с Большим Братом. Telegram и MTProxy](https://habr.com/ru/articles/1018740/) — обзор реализаций MTProxy, упомянут telemt.
- Habr: [Партизанский Telegram: прокси-невидимка под видом онлайн-магазина](https://habr.com/ru/articles/995102/) — концепт fallback-проксирования.
- Habr: [Повышаем стабильность Telegram: MTProxy с Fake TLS](https://habr.com/ru/articles/994934/) — комментарии о handshake timeouts в РФ.
- GitHub: `forhsd/telemt` (Rust source).
