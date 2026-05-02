---
title: Session freezing
aliases: ["Замораживание сессии", "Замораживание после 16 KB"]
type: concept
layer: transport
status: working
status_as_of: 2026-05-02
risk: low
prerequisites: ["[[TCP]]", "[[TCP — состояния]]"]
related: ["[[ТСПУ]]", "[[xHTTP]]"]
sources: [src-02, src-09]
tags: [networking, rf-circumvention, concept]
---
# Session freezing — «замораживание» TCP-сессии

## TL;DR
Особый mode DPI-блокировки: вместо **RST** (явного разрыва) ТСПУ **silently** перестаёт пропускать пакеты в подозрительной сессии. Триггер: **превышение 16 KB** переданного объёма (src-09) или длительная idle-active flow. Клиент видит «зависание» — keep-alive проходит, но реальные данные не доходят. Hard to detect, hard to react automatically — **самая коварная** форма блокировки.

## Какую проблему решает (с точки зрения регулятора)
- **RST-блокировка** легко детектируется клиентом → переключение на резерв.
- **Session freezing** выглядит как «провайдер тормозит» — пользователь винит сеть, не сторону блокировки.
- Время-к-детекции у обходчика растёт → больше «успешных» сессий с точки зрения метрик регулятора.

## Как работает

**Триггеры (известные):**
1. **16 KB transfer threshold** (src-09) — после первых 16 KB пакеты тихо дропаются.
2. **Long-lived TCP без HTTP-семантики** (src-02) — например, чистый VLESS+TLS без HTTP-обёртки.
3. **Behavioral anomaly** — паттерн ML-классификатора отнес поток к VPN.

**Что видит клиент:**
- TCP-handshake: OK.
- Первые 16 KB: проходят.
- Дальше: writes возвращают success (ACK на TCP-уровне), но **payload не достигает peer'а**.
- TCP-keepalive проходит (small probe).
- Application-timeout срабатывает после ~30 с.
- Reconnect: новый handshake, опять 16 KB success, freeze.

**Почему 16 KB:**
- TLS handshake обычно занимает ~5-10 KB (cert chain, ServerHello).
- 16 KB — порог, после которого «handshake закончен и идёт application data» → достаточный sample для классификации.

## Где ломается / почему может не работать (с точки зрения обходчика)
Решения:
1. **xHTTP packet-up** ([[xHTTP]]): каждое sendto = новый POST → не «длинная» сессия для DPI → нет 16 KB threshold для **одной** TCP-connection.
2. **Reconnect after N KB**: VPN-клиент сам разрывает и пересоздаёт каждые ~10 KB.
3. **vnext через РФ-мост**: «короткая» сессия клиент↔мост; bridge↔exit может быть длинной, но это уже в whitelist-AS.
4. **Multiple parallel connections** (multiplexing): много коротких вместо одной длинной.
5. **QUIC/UDP**: stateless transport избегает TCP-flow ассоциации.

## Минимальный сценарий диагностики
```bash
# Replicate freezing:
curl https://blocked-vpn-ip.com:443/ --output /dev/null -w '%{size_download}\n' -m 60
# Если ~16384 bytes (16 KB) — match (freeze).
# Если timeout/refused — другая блокировка.
# Если full download — нет блокировки.
```

`tcpdump` со стороны клиента покажет, что после 16 KB ACKs приходят, но данные больше не передаются.

## Связи
- **Базируется на:** [[TCP]] (что замораживается), [[TCP — состояния]] (state ESTABLISHED при заморозке), [[ТСПУ]] (механизм).
- **Соседи по уровню:** **active probing** (другая DPI-стратегия), [[uTLS]] (один из факторов классификации).
- **Используется в:** мотивация для [[xHTTP]], [[vnext-цепочка]].

## Источники
- src-02 (явное упоминание), src-09 (16 KB threshold).
