---
title: JA3 / JA4 fingerprinting
aliases: ["JA3", "JA4", "TLS fingerprint"]
type: concept
layer: security
status: working
status_as_of: 2026-05-02
risk: low
prerequisites: ["[[TLS — рукопожатие]]"]
related: ["[[uTLS]]", "[[Active probing]]", "[[DPI-фильтрация в РФ]]"]
sources: [src-08, src-10]
tags: [networking, rf-circumvention, concept]
---
# JA3 / JA4 fingerprinting

## TL;DR
Метод **классификации TLS-клиента** по параметрам ClientHello: cipher-suites, extensions, elliptic curves, EC point formats, ALPN, signature algorithms.
- **JA3** (Salesforce, 2017) — MD5 от `(SSLVersion,Ciphers,Extensions,EllipticCurves,EllipticCurvePointFormats)`. 32-символьный hash.
- **JA4** (FoxIO, 2023) — расширение JA3: разнесены TLS-fingerprint, HTTP-fingerprint, TCP-fingerprint, x509-fingerprint; **детерминированный** (не MD5), поддаётся ручному чтению.

В РФ: ТСПУ использует JA3/JA4 для детекции «не-браузерных» TLS-клиентов (Xray, OpenVPN, plain Go-stdlib). Главная защита — [[uTLS]].

## Какую проблему решает (для DPI)
Видимо «обычный TLS» — но клиент-библиотека Go/Rust/OpenSSL имеют свои уникальные комбинации параметров. Браузер (Chrome, Firefox) — другую. Если ClientHello-hash совпадает с одной из 5–10 «легитимных браузерных» — пропускаем; иначе подозрение.

## Как работает

### JA3
Hash-формула:
```
JA3 = MD5(
  SSLVersion + "," +
  Ciphers (joined by "-") + "," +
  Extensions (joined by "-") + "," +
  EllipticCurves (joined by "-") + "," +
  EllipticCurvePointFormats (joined by "-")
)
```

Пример (Chrome 120 на macOS):
```
771,4865-4866-4867-49195-...,0-23-65281-10-11-...,29-23-24,0
→ JA3: cd08e31494f9531f560d64c695473da9
```

### JA4
Структура: `q13d1516h2_8daaf6152771_b186095e22b6`:
- `q13` — QUIC, TLS 1.3, не-domain.
- `1516` — 15 cipher-suites, 16 extensions.
- `h2` — ALPN h2.
- `8daaf6152771` — hash cipher-suite list.
- `b186095e22b6` — hash extension list.

JA4 разделяет компоненты так, что можно изменять только некоторые без слома идентификации.

### JA4S, JA4H, JA4X, JA4T, JA4SSH
JA4 — это семейство:
- **JA4S** — server-side ServerHello fingerprint.
- **JA4H** — HTTP-fingerprint (headers).
- **JA4X** — X.509 сертификат.
- **JA4T** — TCP-options fingerprint.
- **JA4SSH** — SSH protocol.

## Где используется (для DPI)
- **CDN/WAF** (Cloudflare, Akamai) — для anti-bot.
- **ТСПУ / DPI в РФ** — для детекции нелегитимного TLS-stack (src-08).
- **IDS/IPS** (Suricata, Zeek) — встроенные правила.

## Где ломается / способы обхода
- **uTLS** ([[uTLS]]) — Go-библиотека, **полностью имитирует** JA3-fingerprint конкретного браузера. Используется в Xray, V2Ray, Hysteria.
- **uTLS требует обновлений:** при выходе нового Chrome — старые fingerprint'ы устаревают. Сообщество ledит эту гонку.
- **JA4 устойчивее:** JA4 разносит параметры так, что imperfect-имитация (часть параметров правильная, часть нет) даёт уникальный hash → детектируется.
- **Active probing после JA-match:** даже если JA3 chrome — DPI может подтвердить через [[Active probing]].

## Связи
- **Базируется на:** [[TLS — рукопожатие]] (источник параметров).
- **Используется в:** [[DPI-фильтрация в РФ]] (как метод), [[Active probing]] (комбинация).
- **Обходится через:** [[uTLS]] (mimicry).
- **Соседи по уровню:** **JARM** (более сложный server-fingerprint), **CYU** (cipher-suite-only).

## Источники
- src-08, src-10.
