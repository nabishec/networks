---
title: PingZen
aliases: ["PingZen MTProxy checker"]
type: tool
layer: application
status: working
status_as_of: 2026-05-02
risk: low
prerequisites: ["[[MTProxy и FakeTLS]]", "[[TLS — рукопожатие]]", "[[HMAC]]"]
related: ["[[mtg]]", "[[PB8 — MTProxy + FakeTLS]]", "[[hyperion-cs DPI-checker]]"]
sources: [src-10]
tags: [networking, rf-circumvention, tool]
---
# PingZen — health-check для MTProxy

## TL;DR
**PingZen** — SaaS-мониторинг с поддержкой **22+ протоколов**, среди которых уникальная для индустрии возможность — **полноценный health-check MTProxy/FakeTLS**. Обычные uptime-сервисы (UptimeRobot, Pingdom, Hetrix, Better Stack) MTProxy чекать **не умеют** — они проверяют только TCP-connect, который для MTProxy **бесполезен**: сокет может быть жив, а protocol-stack уже мёртв (FakeTLS-handshake не завершается). PingZen ходит **полным путём** реального Telegram-клиента — отсюда упоминание в src-10 как ключевого инструмента «гонки вооружений» с ТСПУ.

## Что делает
- **22 протокола:** ICMP/TCP/UDP/HTTP(s)/DNS/SMTP/POP3/IMAP/FTP/Heartbeat/Playwright + **MTProxy** + SOCKS5 + PageSpeed.
- **MTProxy-checker** (src-10) — точный путь Telegram-клиента:
  1. **TCP-connect** на `host:port`. Распознаёт connection-refused / DNS-failure / timeout.
  2. **TLS 1.3 ClientHello** — формирует ровно **517 байт**, как настоящий Telegram (включая HMAC в random-поле через secret).
  3. **Validate ServerHello** + HMAC-валидация ответа.
  4. **Obfuscated2-init** в TLS-record + ожидание post-handshake-ответа.
  5. Если шаги 1-4 OK — статус **UP** (= реальный пользователь подключится).
- В отличие от TCP-ping: TCP-up + MTProxy-down случается **постоянно** (DPI режет post-handshake без RST).
- Архитектура: модульный `BaseHealthChecker` interface, asyncio, метрики в PostgreSQL + VictoriaMetrics.

## Использование
- SaaS: https://pingzen.io (или аналог).
- **MCP-сервер** (Habr-1015072) — 126 инструментов через Claude Code/MCP-клиент.
- Для self-host есть API + push-heartbeat (для NAT-behind серверов).

## Зачем это владельцу MTProxy
- Заметить deadhost **до** жалоб пользователей — Telegram-клиент даёт generic «Proxy unavailable», без диагностики.
- Отличить **L3-блокировку** (TCP-down) от **L7-блокировки ТСПУ** (TCP-up + protocol-down).
- Алерты на flaky proxy: уход в «частично рабочее» — ранний сигнал, что IP попал в маркировку.

## Зачем это «защитникам» (с другой стороны)
- src-10: PingZen-подобные мониторы используются **для пополнения blacklist'ов** — публичные MTProxy быстро попадают в детектор. Любой публичный MTProxy с известным `tg://`-link'ом проверяется автоматически.
- Если **частный** прокси — IP не «светится» в публичных списках → дольше живёт.

## Где ломается
- **Точность health-check'а ограничена** — PingZen не моделирует **полный** жизненный цикл сессии (только handshake + первый response). DPI с **post-handshake ML-detection** (src-10) может пропустить handshake, но прервать сессию через 5-10 секунд.
- **False-positive UP** при TCP-up + handshake-up + DPI-cut-after-30s.
- **Rate-limit:** агрессивная проверка с одного IP → cloud-провайдер может classify его как scanner.

## Связи
- **Проверяет:** [[MTProxy и FakeTLS]], [[mtg]] серверы.
- **Соседи по уровню:** Uptime Kuma (open-source, **без** MTProxy-checker), Better Stack, Hetrix.
- **Концептуально близко:** [[hyperion-cs DPI-checker]] — специализированные DPI-диагностики; [[RealiTLScanner]] — для VLESS-Reality target'ов.

## Где взять
- SaaS: https://pingzen.io
- Документация по MCP-server: см. публикации команды на Habr.

## Источники
- src-10 (PingZen упомянут как ключевой инструмент мониторинга MTProxy в РФ-2026).
- Habr: [22 протокола мониторинга в PingZen: от пинга до Playwright-сценариев](https://habr.com/ru/articles/1010322/) — обзор возможностей.
- Habr: [Мониторинг, который не бесит: почему мы перестали использовать Uptime Kuma и написали свой SaaS с поддержкой UDP/ICMP](https://habr.com/ru/articles/1002300/) — мотивация и архитектура.
- Habr: [Как мы превратили PingZen в MCP-сервер с 126 инструментами](https://habr.com/ru/articles/1015072/) — MCP-интеграция.
