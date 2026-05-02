---
title: VLESS-Reality
aliases: ["Reality", "XTLS-Reality"]
type: technique
layer: application
status: working
status_as_of: 2026-05-02
risk: medium
prerequisites: ["[[TLS — рукопожатие]]", "[[X.509 сертификаты]]", "[[HTTPS]]"]
related: ["[[XTLS-Vision]]", "[[xHTTP]]", "[[uTLS]]", "[[Active probing]]"]
sources: [src-02, src-03, src-06, src-08]
tags: [networking, rf-circumvention, technique]
---
# VLESS-Reality

## TL;DR
Прокси-протокол поверх TLS, **маскирующийся под легитимный сторонний сайт** (например, microsoft.com): handshake идёт **с реальным целевым сертификатом**, DPI видит «настоящий» TLS к настоящему серверу. На сервере специально настроенный Xray перехватывает соединение **до отправки данных**, проверяет криптомаркер от клиента — если совпало, направляет в свой VPN-канал; если нет — проксирует к настоящему сайту. Для «active probing» атакующего (DPI-сканера) сервер выглядит абсолютно как microsoft.com.

## Какую проблему решает
До Reality: классический VLESS+TLS использовал **свой** сертификат на своём домене → DPI ловил по SNI или по аномалии TLS-сертификата. **Domain fronting** через CDN позволял прятать SNI, но Cloudflare и др. начали закрывать. Reality снимает все три проблемы: SNI настоящий, сертификат настоящий, fingerprint настоящий — потому что **handshake реально идёт к чужому серверу** и только потом перенаправляется.

## Как работает

```mermaid
sequenceDiagram
  participant C as Client
  participant DPI as DPI/ТСПУ
  participant S as Reality-server
  participant T as Target (microsoft.com)
  C->>S: TLS ClientHello (SNI: microsoft.com) + secret в random
  S->>T: TLS handshake forward (как обычный proxy к target)
  T->>S: ServerHello + cert (от microsoft.com)
  S->>C: forward ServerHello (cert настоящий!)
  Note over S: проверяет secret в client random; совпало → switch на VPN
  C->>S: VLESS-данные внутри TLS
  Note over DPI: видит handshake к microsoft.com<br/>cert валидный, fingerprint настоящий
```

**Главные элементы:**
- **x25519-ключевая пара** на сервере. Public key вкладывается в client-config.
- Клиент генерирует случайный `random` в ClientHello и **встраивает короткий secret** через ECDH с server public.
- Сервер декодирует — если secret валиден → это «свой клиент» → переключение на VPN; иначе — реальная проксировка к target.
- **uTLS на клиенте** — имитирует fingerprint реального браузера (Chrome, Firefox, Safari).

## Где ломается / почему может не работать
- Target-сайт должен **поддерживать TLS 1.3** + ALPN h2 + ECH-неподдержка (классический). Целевые: microsoft.com, github.com, apple.com — стандарт.
- В **РФ-whitelist-провайдерах** target-IP должен быть в whitelist. Если microsoft.com заблокирован — Reality не работает.
- **src-06 (декабрь 2025):** в РФ некоторые DPI начали ловить Reality по **NewSessionTicket-fingerprint** — нужен Xray ≥ v25.12.8 (src-02).
- **Active probing:** DPI может «постучаться» сам и проверить — Reality защищает (отвечает как target), но ошибки конфига могут раскрыть.

## Минимальный пошаговый сценарий
См. [[PB7 — basic VLESS-Reality с нуля]].

## Что нужно
- **Сервер:** VPS вне РФ (или внутри для cascade с PB2/PB5), Xray-core ≥ v25.12.8.
- **Target-сайт** для маскировки (только TLS 1.3, желательно популярный).
- **x25519-ключевая пара** (`xray x25519`).
- **Клиент:** Hiddify-Next, Nekobox/Nekoray, v2rayN/NG, Streisand, Shadowrocket, Happ.
- **Свой домен НЕ нужен** (в отличие от классического TLS-VPN).

## Связи
- **Базируется на:** [[TLS — рукопожатие]], [[X.509 сертификаты]], [[Диффи-Хеллман]] (ECDH через x25519).
- **Соседи по уровню:** [[XTLS-Vision]] (flow), [[xHTTP]] (transport-обёртка).
- **Используется в:** [[PB2 — vnext-цепочка через РФ-мост]], [[PB3 — 4-уровневая архитектура за 265₽]], [[PB7 — basic VLESS-Reality с нуля]].
- **Противопоставляется:** classical VLESS+TLS со своим сертификатом — DPI его теперь ловит; [[CDN-фронтинг]] — другая школа маскировки.

## Источники
- src-02, src-03, src-06, src-08 — см. [[_sources]].
