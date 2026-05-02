---
title: MTProxy и FakeTLS
aliases: ["MTProxy", "FakeTLS", "MTProto", "Obfuscated2"]
type: technique
layer: application
status: partial
status_as_of: 2026-05-02
risk: medium
prerequisites: ["[[TLS — рукопожатие]]", "[[HMAC]]", "[[Хеш-функции]]"]
related: ["[[VLESS-Reality]]"]
sources: [src-10]
tags: [networking, rf-circumvention, technique]
---
# MTProxy и FakeTLS

## TL;DR
**MTProxy** — официальный proxy-протокол Telegram (на основе MTProto). Эволюция:
- **Obfuscated2 (DD-поколение):** случайный поток с HMAC-метаданными в начале.
- **FakeTLS (EE-поколение):** маскирует первые байты как **TLS 1.3 ClientHello** + HMAC-SHA256 hidden-handshake. DPI видит «обычный TLS».

**Статус по типу маскировки** (на 2026-05-02, src-10):

| Вариант | Status | Почему |
|---|---|---|
| MTProxy без маскировки / Obfuscated2 | **broken** | DPI ловит post-handshake-pattern почти сразу |
| FakeTLS обычный (произвольный SNI) | **partial** | ТСПУ детектирует через **post-handshake behavior** + ML; срок жизни IP — дни-недели |
| FakeTLS «золотые прокси» (SNI под yandex.ru / gosuslugi.ru / sberbank.ru) | **working** | trusted-домен в SNI; ТСПУ не блокирует, чтобы не сломать сами эти ресурсы |

Frontmatter-status `partial` — обобщённая оценка для default-варианта; «золотые» отдельно отмечены в [[applied-rf-status]].

## Какую проблему решает
- Telegram блокировки в РФ периодически (2018, 2022+) → нужен прокси.
- MTProxy — **встроенная** возможность Telegram-клиента (без отдельного VPN).
- FakeTLS обходит **простой SNI/fingerprint-DPI**.

## Как работает

**FakeTLS handshake:**
1. Клиент шлёт **TLS-1.3-look-alike ClientHello** с HMAC-secret в random-поле.
2. Сервер проверяет HMAC через secret → если совпало, обрабатывает как MTProxy.
3. Шлёт **TLS-look-alike ServerHello** + HMAC.
4. Дальше — **MTProto-encrypted** payload в TLS-record-обёртках.
5. Если HMAC не совпадает — сервер ведёт себя как обычный HTTPS-сайт (proxy наружу).

**HMAC-SHA256 hidden-handshake:**
- secret = pre-shared между клиентом и сервером (в `tg://proxy?...`-link'е).
- HMAC-метаданные **встроены** в TLS-random — DPI без знания secret не отличит от обычного TLS.

## Где ломается / почему может не работать
- **Post-handshake detection** (src-10): ТСПУ детектирует **поведенческие** аномалии: длительность connection, packet-sizes, ML-классификатор.
- **Reply-attack-resistance:** MTProxy уязвим к detection через flow-statistics.
- **Прокси с известными IP** быстро попадают в blacklist через PingZen-like мониторинг.
- **«Золотые прокси»** маскируются под trusted-домены (yandex.ru) — пока работают, но это юридически спорно.

## Минимальный пошаговый сценарий
**Сервер (mtg, Go-реализация):**
```bash
mtg simple-run --bind 0.0.0.0:443 --secret <hex-secret>
# или с FakeTLS-доменом для маскировки:
mtg simple-run --secret ee<...>www.google.com.hex
```

**Клиент:** Telegram → Settings → Data and Storage → Proxy → Add Proxy → MTProxy → server, port, secret.

## Что нужно
- VPS с public IP (порт 443 или другой).
- mtg / telemt / official MTProxy (Telegram repo).
- HMAC-secret (генерируется автоматически).

## Связи
- **Базируется на:** [[TLS — рукопожатие]] (маскировка), [[HMAC]] (auth), [[Хеш-функции]].
- **Соседи по уровню:** [[VLESS-Reality]] (концептуально похожий fakeness), Trojan (TLS-based proxy).
- **Используется в:** [[PB8 — MTProxy + FakeTLS]].
- **Противопоставляется:** простой HTTPS-proxy — нет маскировки.

## Источники
- src-10.
