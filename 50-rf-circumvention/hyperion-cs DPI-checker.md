---
title: hyperion-cs DPI-checker
aliases: ["hyperion-cs", "DPI-checkers", "TCP 16-20 checker"]
type: tool
layer: cross-layer
status: working
status_as_of: 2026-05-02
risk: low
prerequisites: ["[[DPI-фильтрация в РФ]]", "[[ТСПУ]]", "[[Session freezing]]", "[[AS-level whitelist]]", "[[TCP]]", "[[TLS — рукопожатие]]"]
related: ["[[Белые списки]]", "[[PB4 — диагностика whitelist]]", "[[PingZen]]", "[[BGP]]"]
sources: [src-09]
tags: [networking, rf-circumvention, tool, diagnostic]
---
# hyperion-cs DPI-checker

## TL;DR
Набор **браузерных и desktop-чекеров** на https://hyperion-cs.github.io/dpi-checkers/, диагностирующих **специфические эффекты РФ-DPI**: 16-20 KB session-freezing ([[Session freezing]]), AS-level whitelist ([[AS-level whitelist]]), SNI-фильтрация. В отличие от обычного `ping`/`curl`, эти тулы измеряют **именно те аномалии**, что характерны для ТСПУ-2025+ — отсюда упоминание в src-09 как «диагностический инструмент №1» при анализе пострадавших AS.

## Что входит
Репозиторий `hyperion-cs/dpi-checkers` содержит четыре утилиты:

### 1. RU :: DPI-CH (Comprehensive Checker)
- **Desktop**, Go, Windows/macOS/Linux.
- Не ограничен sandbox'ом браузера → может проверять **SNI-filtering** (нужен сырой TCP/TLS).
- Гибкая конфигурация: список доменов, ASN-фильтр, custom-timeout.
- «Power tool» для глубокого анализа: какие из 1M Tranco-доменов проходят, какие SNI блокируются.

### 2. RU :: TCP 16-20
- **Browser-based**, JS.
- Тестирует именно **«механику отсечки в 16-20 KB»** ([[Session freezing]]) — отправляет ≥20 KB к различным provider-hosted сервисам и засекает, на каком байте обрывается ответ.
- Параметры: timeout (default 15000 ms), custom-hosts.
- Главный диагностический инструмент: проверить, **активирован ли** session-freezing у провайдера прямо сейчас.

### 3. RU :: IPv4 Whitelisted Subnets
- Browser, JS.
- Сканирует CIDR-ranges → отмечает «прошедшие» подсети.
- Кеширует результат, экспорт CSV.
- Назначение: если ты на mobile-whitelist — **за час** выяснить, какие /24 ещё проходят (чтобы заказать VPS в правильной AS).

### 4. RU :: TCP 16-20 DWC (Domain Whitelist Checker)
- Python + curl.
- Перебирает домены из списка → определяет, какие «протаскивают» трафик через session-freezing-DPI (т.е. domains, которые ТСПУ не режет на 16 KB).
- Полезен для **подбора SNI-mask** в [[VLESS-Reality]] / [[CDN-фронтинг]].

## Использование (TCP 16-20 web-checker)
1. Зайти на https://hyperion-cs.github.io/dpi-checkers/ru/tcp-16-20/
2. Нажать **Start**.
3. Браузер шлёт серию запросов к набору test-серверов; результат отображается per-host:
   - **OK** — провайдер не режет, ответ ≥20 KB пришёл целиком.
   - **CUT @ 16384** — характерная сигнатура session-freezing-DPI.
   - **TIMEOUT** — что-то жёстче или вообще IP-blocked.
4. Сохранить лог.

## Использование (DPI-CH desktop)
```bash
# Скачать релиз для своей ОС с https://github.com/hyperion-cs/dpi-checkers/releases
./dpi-ch --domains tranco-1m.txt --asn-filter "RU" --out report.csv
```

## Где ломается
- **Browser sandbox** не даёт делать raw-TCP/SNI-tests → web-checker не видит **TLS-уровень** (только HTTP-cutoff). Для SNI-фильтрации нужен desktop-вариант.
- **Не показывает причину** — просто фиксирует «обрывается тут». Чтобы понять, **что именно** режет (DPI-stateful vs CGNAT-timeout), нужен Wireshark + анализ TCP-флагов.
- **False-positive** на overloaded test-server'ах — повторить через 10 минут.
- **Не моделирует** реальный VPN-трафик — только generic HTTPS. Для проверки конкретного протокола (Reality / xHTTP) использовать [[PingZen]] / тестовый клиент.

## Связи
- **Диагностирует:** [[Session freezing]] (16-20 KB cutoff), [[AS-level whitelist]], [[Белые списки]] (subnet whitelist), [[SNI-фильтрация]].
- **Дополняет:** [[PB4 — диагностика whitelist]] (общий playbook диагностики).
- **Соседи по уровню:** `cheburcheck.ru` (публичный whitelist-checker), `dpi-detector` (open-source self-test).

## Где взять
- Web: https://hyperion-cs.github.io/dpi-checkers/
- GitHub: https://github.com/hyperion-cs/dpi-checkers

## Источники
- src-09 (упомянут как ключевой инструмент при разборе «72 AS allowed / 391 AS damaged»).
- Habr: [РКН создали белый список для 72 AS, но пострадали 391 AS (>225 млн IP адресов)](https://habr.com/ru/articles/997088/) — в статье ссылка на TCP-16-20-checker как способ воспроизвести эффект.
- Habr: [Белые списки добрались до Москвы: изучаем механику «отсечки» в 16 килобайт](https://habr.com/ru/articles/1008164/) — детали 16 KB session-freezing, измеряемые этими чекерами.
