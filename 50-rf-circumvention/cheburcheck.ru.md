---
title: cheburcheck.ru
aliases: ["cheburcheck", "Чебурчек"]
type: tool
layer: cross-layer
status: working
status_as_of: 2026-05-02
risk: low
prerequisites: ["[[Белые списки]]", "[[DPI-фильтрация в РФ]]", "[[TLS — рукопожатие]]", "[[DNS]]"]
related: ["[[hyperion-cs DPI-checker]]", "[[dpi-detector]]", "[[PB4 — диагностика whitelist]]", "[[SNI-фильтрация]]"]
sources: [src-04, src-01]
tags: [networking, rf-circumvention, tool]
---
# cheburcheck.ru

## TL;DR
**Публичный веб-checker** с базой знаний по whitelist-режимам РФ-операторов: `cheburcheck.ru/kb/whitelist`. Открываете страницу — браузер делает серию HTTPS-запросов с разными SNI к подконтрольным IP, и сайт показывает, **какие техники blocking'а** активны на вашем текущем подключении (mobile или домашнее).

«Чебурнет» в названии — отсылка к концепту изолированного русского интернета («Чебурашка»).

## Какую проблему решает
Нужен быстрый «один клик» способ узнать, в каком режиме сейчас провайдер: whitelist / blacklist / mixed. В отличие от [[dpi-detector]] (CLI, нужно ставить) — открывается с любого устройства, включая мобильный.

## Как работает

Сайт делает в фоне (через JavaScript-`fetch`):
1. **HTTPS на «свой» IP с SNI=cheburcheck.ru** — на whitelist-mobile timeout @ 16 КБ ([[Session freezing]]).
2. **HTTPS на тот же IP с SNI=ok.ru** (whitelisted) — мгновенно проходит, демонстрируя что фильтрация именно по SNI.
3. **HTTPS на популярный non-whitelist IP** (Hetzner / OVH) — timeout = blacklist-AS.
4. **DNS-запросы** к 1.1.1.1, 8.8.8.8, 77.88.8.8 — какие резолверы достижимы.

Автор статьи [src-04](https://habr.com/ru/articles/1008164/) подробно описал механику через cheburcheck.ru: «при запросе с SNI cheburcheck.ru — таймаут на 16 КБ, но при том же IP с подменой SNI на ok.ru данные проходят мгновенно».

## Что показывает на странице
- Таблица: `Тест → Результат → Объяснение`.
- Подсказки по обходу для конкретного режима.
- База знаний `/kb/whitelist` — что такое whitelist, как устроен, чем отличается от blacklist.

## Где ломается
- Если провайдер **дропает HTTPS к cheburcheck.ru** на L3 (IP-blacklist) — сайт не открывается вообще. Но это уже сам по себе диагностический сигнал.
- Сайт может попасть в whitelist-blacklist'ы РКН — в любой момент ресурс может стать недоступным с конкретного оператора.
- В отличие от [[dpi-detector]], не показывает JA3-fingerprint-блокировки и UDP-throttling.

## Связи
- **Соседи по уровню:** [[hyperion-cs DPI-checker]] (более технический, по конкретным DPI-эффектам), [[dpi-detector]] (CLI-вариант).
- **Используется в:** [[PB4 — диагностика whitelist]] как одна из проверок.
- **Что детектирует:** [[SNI-фильтрация]], [[Session freezing]], [[Белые списки]], [[AS-level whitelist]].

## Источники / Дальше читать
- [cheburcheck.ru/kb/whitelist](https://cheburcheck.ru/kb/whitelist) — сам инструмент + база знаний.
- [src-04 — Белые списки добрались до Москвы](https://habr.com/ru/articles/1008164/) — детальный разбор методики через cheburcheck.
- [src-01 — Белые списки: 6 способов обхода](https://habr.com/ru/articles/1027276/) — упомянут как один из публичных checker'ов.
