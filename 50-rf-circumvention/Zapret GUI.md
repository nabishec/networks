---
title: Zapret GUI
aliases: ["Zapret", "zapret-discord-youtube", "Zapret 2 GUI"]
type: tool
layer: cross-layer
status: working
status_as_of: 2026-05-02
risk: medium
prerequisites: ["[[DPI-фильтрация в РФ]]", "[[ТСПУ]]", "[[TCP]]"]
related: ["[[VLESS-Reality]]", "[[AmneziaWG]]", "[[Session freezing]]"]
sources: [src-04]
tags: [networking, rf-circumvention, tool]
---
# Zapret / Zapret GUI

## TL;DR
**Zapret** (`bol-van/zapret`) — open-source CLI-инструмент **локального** обхода DPI: модифицирует исходящие TCP/TLS/QUIC-пакеты (фрагментация, fake-headers, TTL-fake, oob-byte) так, что DPI **не успевает** их классифицировать. Не VPN — не уносит трафик в туннель, просто «портит» сигнатуры на лету через NFQUEUE/iptables. Работает **только** против тех DPI-классов, что чувствительны к таким маскировкам (т.е. ТСПУ-старые правила, не современный whitelist). **GUI-варианты** (Zapret GUI, GoodbyeDPI Launcher) упрощают подбор стратегии под конкретного оператора.

## Что делает
- Перехватывает исходящий трафик через **NFQUEUE** (Linux) / **WinDivert** (Windows).
- Применяет **DPI-evasion-стратегию**: TCP segmentation, TLS-record split, packet-up fake, TTL truncation.
- Сервер не нужен — **локальный** инструмент на клиенте.
- В [src-04](_sources.md#src-04) упомянут как способ подобрать стратегию для конкретного провайдера на mobile-whitelist (но эффективен **только** где DPI ещё «черносписочный», не whitelist-режим).

## Использование (Linux)
```bash
git clone https://github.com/bol-van/zapret.git && cd zapret
./install_easy.sh
# Скрипт спрашивает: фрагментация / fake-headers / hostlist
# и применяет iptables + NFQUEUE правила.
systemctl enable --now zapret
```

## Использование (Windows, GUI)
- **GoodbyeDPI Launcher** — официально-нейтральный GUI-обёрткa над `goodbyedpi.exe` (родственный проект). Подбирает стратегию через `--blacklist`-listы.
- **B4** — Go + web-UI; auto-discovery стратегии под конкретную сеть.
- Запуск: «Run as Administrator» → выбрать профиль (YouTube/Discord/All) → Apply.

## Подбор стратегии
DPI у разных операторов реагирует по-разному. Стандартный workflow (src-04):
1. Выбрать профиль `--ttl-fake` для самых простых DPI.
2. Если не работает — `--split-pos=2` (фрагментация TLS-record после 2 байт).
3. `--wssize 1:6` — fake-headers + window-size manipulation.
4. Если ничего — переходить на VPN (Reality / AmneziaWG).

## Где ломается / **важное предупреждение**
- **Не работает на whitelist-режиме** (mobile-РФ-2026): DPI не «классифицирует» — он просто блокирует **всё** кроме разрешённого. Никакая packet-modification не пропустит трафик к не-whitelisted IP/SNI.
- **Гонка вооружений:** ТСПУ обновляется → стратегии устаревают за месяцы.
- **GUI-форки бывают вредоносными.** В Habr-1015380 разобран форк «Zapret 2 GUI», который под видом обхода:
  - отключает Windows Defender через registry-modifications;
  - устанавливает root-CA для MITM HTTPS (для DPI-bypass cert **не нужен** — это явный red flag);
  - шлёт телеметрию каждые 30 минут на admin-аккаунт.
  - Скомпилированный .exe **отличался** от публичного исходного кода.
- **Правило:** ставить **только** оригинальный `bol-van/zapret` или верифицированные форки с прозрачным CI.

## Связи
- **Атакует:** [[DPI-фильтрация в РФ]], [[ТСПУ]] (старые правила).
- **Соседи по уровню:** GoodbyeDPI (Windows-аналог), B4 (Go + web-UI), Spoof-DPI.
- **Не заменяет:** [[VLESS-Reality]] / [[AmneziaWG]] — на whitelist-режиме нужен туннель.
- **Диагностика эффективности:** [[hyperion-cs DPI-checker]] (TCP 16-20-checker).

## Где взять
- GitHub: https://github.com/bol-van/zapret (canonical, audited).
- Windows: GoodbyeDPI — https://github.com/ValdikSS/GoodbyeDPI

## Источники
- src-04 (Zapret GUI как способ подбора DPI-стратегии под mobile-оператора).
- Habr: [GUI ценой приватности: разбор вредоносного форка Zapret 2 GUI](https://habr.com/ru/articles/1015380/) — анализ malware-форка, отключение Defender, MITM-cert.
- Habr: [B4 — обход DPI с веб-интерфейсом](https://habr.com/ru/articles/982498/) — Go-вариант с auto-discovery стратегий.
- Habr: [Домашний DPI, или как бороться с провайдером его же методами](https://habr.com/ru/articles/548110/) — теоретическая база Zapret.
