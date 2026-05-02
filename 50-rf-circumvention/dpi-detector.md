---
title: dpi-detector
aliases: ["dpi-detector", "Runnin4ik/dpi-detector"]
type: tool
layer: cross-layer
status: working
status_as_of: 2026-05-02
risk: low
prerequisites: ["[[DPI-фильтрация в РФ]]", "[[Session freezing]]", "[[TLS — рукопожатие]]", "[[Брандмауэр]]"]
related: ["[[hyperion-cs DPI-checker]]", "[[cheburcheck.ru]]", "[[PB4 — диагностика whitelist]]", "[[DNS]]"]
sources: [src-09, src-04]
tags: [networking, rf-circumvention, tool]
---
# dpi-detector (Runnin4ik)

## TL;DR
**Open-source-CLI-чекер** от `Runnin4ik` (GitHub), определяющий **класс DPI-эффектов** на текущем подключении: 16-КБ-отсечка, SNI-фильтрация, IP-blacklist, AS-whitelist, port-blocking. В отличие от веб-чекеров ([[hyperion-cs DPI-checker]], [[cheburcheck.ru]]), работает **локально** — без отправки трафика на чужой сервер.

## Какую проблему решает
Перед выбором техники обхода нужно понять **режим** провайдера: blacklist / whitelist / mixed. dpi-detector — single-binary, который запускает серию диагностических TCP/TLS/UDP-проб и классифицирует ответ.

## Как работает

### Тесты внутри
1. **TCP к whitelist-IP** (Yandex / VK) с разными SNI: проверка SNI-фильтра.
2. **TCP к non-whitelist-IP** (Hetzner test-IP): проверка IP-blacklist'а.
3. **HTTPS с большим payload** (>30 КБ): проверка [[Session freezing|16-КБ-отсечки]].
4. **TLS handshake с разными ClientHello fingerprint** (Chrome / curl / Go-stdlib): проверка JA3-блокировки.
5. **UDP/443**: проверка QUIC-throttling.
6. **DNS через 8.8.8.8 / 1.1.1.1 / 77.88.8.8**: какие resolver'ы доступны.

### Вывод
Табличка с 6-7 строками:
```
[OK]    DNS Yandex  77.88.8.8       2ms
[FAIL]  DNS Google   8.8.8.8        timeout (whitelist-DNS)
[OK]    HTTPS Yandex SNI=ya.ru      200 OK
[FAIL]  HTTPS Hetzner SNI=test.com  timeout @ 16384 bytes (session-freezing)
[FAIL]  UDP/443 Hetzner             dropped (UDP throttle)
[OK]    Chrome JA3 → Yandex         passed
```

## Минимальный сценарий
```bash
git clone https://github.com/Runnin4ik/dpi-detector.git
cd dpi-detector && go build .
./dpi-detector --verbose
```
Вывод сравнить с ожидаемыми режимами в [[PB4 — диагностика whitelist]].

## Что нужно
- Linux/macOS/Windows + Go 1.21+.
- Прямое подключение (без прокси/VPN) — иначе детектируется `tun0`-маршрут.
- Не запускать **через** существующий VPN — результаты будут смещены.

## Связи
- **Соседние tools:** [[hyperion-cs DPI-checker]] (web-based, более удобный), [[cheburcheck.ru]] (онлайн-checker), [[PingZen]] (специфичен для MTProxy).
- **Используется в:** [[PB4 — диагностика whitelist]] (один из инструментов диагностики).
- **Что детектирует:** [[Session freezing]], [[SNI-фильтрация]], [[AS-level whitelist]], [[Active probing]] (косвенно).

## Источники / Дальше читать
- GitHub: [`Runnin4ik/dpi-detector`](https://github.com/Runnin4ik/dpi-detector).
- [src-09 — РКН и 72 AS](https://habr.com/ru/articles/997088/) — упомянут как часть инструментария исследования.
- [src-04 — Белые списки добрались до Москвы](https://habr.com/ru/articles/1008164/) — методика тестирования (1 млн доменов Tranco).
- Habr: [Определитель типа блокировки сайтов у провайдера](https://habr.com/ru/articles/228305/) — концепт автоматизированной диагностики.
