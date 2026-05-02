---
title: uTLS
aliases: ["uTLS", "TLS fingerprint mimic"]
type: concept
layer: application
status: working
status_as_of: 2026-05-02
risk: low
prerequisites: ["[[TLS — рукопожатие]]"]
related: ["[[VLESS-Reality]]", "[[Active probing]]"]
sources: [src-08]
tags: [networking, rf-circumvention, concept]
---
# uTLS — TLS fingerprint mimicry

## TL;DR
Модифицированная Go-библиотека TLS, позволяющая **имитировать TLS-fingerprint** разных браузеров (Chrome 130, Firefox 120, Safari, …). Стандартный Go-tlsfp детектируется как **«не браузер»** через **JA3/JA4-классификатор** — поэтому Xray/Sing-box/v2ray используют **uTLS** для внешнего вида «как Chrome». Базис всего семейства Reality, xHTTP, и других обходчиков.

## Какую проблему решает
TLS-fingerprint = совокупность параметров ClientHello: cipher-suites, extensions, supported versions, ALPN, signature algorithms, и их **порядок**. Каждый клиент имеет уникальный «отпечаток».
- Реальный Chrome: специфическая последовательность.
- Стандартный Go: своя специфическая, узнаваемая.
- Без uTLS: DPI ловит «это не Chrome → подозрительно».

## Как работает

**Без uTLS:**
```
Client (Go-tls) → ClientHello → JA3: c8a8a04...
DPI: видит JA3 → не в списке популярных браузеров → флаг
```

**С uTLS:**
```
Client (uTLS, mimicking Chrome 130) → ClientHello → JA3: 31ce6b5... (Chrome)
DPI: совпадает с Chrome 130 → пропускает
```

**Профили uTLS:**
- `HelloChrome_120` (Chrome 120 fingerprint).
- `HelloFirefox_120`.
- `HelloIOS_14_0` (Safari iOS).
- `HelloRandomized` (любой случайный из набора).

В Xray-конфиге:
```json
"streamSettings": {
  "tlsSettings": { "fingerprint": "chrome" }
}
```

## Где ломается / почему может не работать
- **Поддержание актуальности:** Chrome обновляется → JA3 меняется. uTLS-fp нужно регулярно обновлять.
- **JA4** (новый, 2023+) — более устойчивый fingerprint, учитывает HTTP/2 settings, ALPN ordering. uTLS постепенно адаптируется.
- **Behavioral fingerprinting** — DPI смотрит не только на handshake, но на **поведение после** (timing, packet sizes). uTLS не маскирует это.
- **TLS 1.3 unification** — все современные браузеры с TLS 1.3 имеют похожий fingerprint, что упрощает mimicry, но также упрощает классификацию anomalies.

## Минимальный пошаговый сценарий
В Xray client-config:
```json
"streamSettings": {
  "security": "reality",
  "realitySettings": { ... },
  "tlsSettings": { "fingerprint": "chrome" }
}
```
Параметр **`fingerprint`** в `tlsSettings` или `realitySettings` указывает uTLS-профиль.

## Связи
- **Базируется на:** [[TLS — рукопожатие]] (механизм mimicry).
- **Используется в:** [[VLESS-Reality]] (всегда нужен), [[XTLS-Vision]], [[Shadowsocks-2022]] (с TLS-обёрткой).
- **Соседи по уровню:** [[Active probing]] (защищает от него), JA3/JA4-классификация.
- **Противопоставляется:** «голый» Go TLS — узнаваем DPI.

## Источники
- src-08.
