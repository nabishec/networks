---
title: Регулирование PSTN — LATA
aliases: ["LATA", "AT&T дивестиция", "LEC", "IXC", "MFJ"]
type: concept
layer: physical
chapter: 2
difficulty: basic
prerequisites: ["[[PSTN — телефонная сеть]]"]
related: ["[[Стандартизация сетей]]"]
tags: [networking, ch02]
---
# Регулирование PSTN — LATA, AT&T дивестиция

## TL;DR
**1984:** US антимонопольный процесс **MFJ** (Modified Final Judgment) разделил **AT&T** на **8 компаний**: один long-distance (AT&T) + 7 RBOC (Regional Bell Operating Companies). Создал понятие **LATA** (Local Access and Transport Area) — географический регион (~164 в США). Внутри LATA — **LEC** (Local Exchange Carrier); между — **IXC** (Inter-Exchange Carrier). Разделение определило архитектуру PSTN на десятилетия.

## Какую проблему решает
До 1984 г. AT&T (Bell System) был **vertically integrated монополист** в США: оборудование, локальная связь, междугородняя, международная — всё одно. Это душило конкуренцию. MFJ разорвал монополию, разделив рынок на сегменты.

Эта структура ушла глубоко в архитектуру: LATA-границы стали техническими, не только юридическими. Понимание этого нужно для разговоров о **carrier-grade** инфраструктуре.

## Как работает

**Структура после MFJ:**

| Уровень | Кто | Что делает |
|---|---|---|
| **Local (intra-LATA)** | **LEC** (Local Exchange Carrier) | звонки внутри LATA |
| **Long-distance (inter-LATA)** | **IXC** (Inter-Exchange Carrier) | между LATA |

**RBOC** (Regional Bell Operating Companies, «Baby Bells»):
- 7 региональных компаний, владеющих local infrastructure: NYNEX, Bell Atlantic, Ameritech, BellSouth, Pacific Telesis, US West, Southwestern Bell.
- Каждая = LEC в своём регионе.

**Слияния позже:**
- Postрядов слияний к 2006: SBC поглотил многих, забрал имя AT&T → новый AT&T (но не тот, что был до 1984).
- Verizon — наследник Bell Atlantic + GTE + NYNEX.

**LATA:**
- ~164 региона в США.
- Звонок A→B внутри LATA → только LEC.
- Звонок A→B между LATA → LEC → IXC → LEC. Каждый получает плату.

**Технические следствия:**
- **POP** (Point of Presence) — точка где IXC соединяется с LEC.
- **Tariffs** — публичные ставки (regulation).
- **Number portability** (с 2003) — можно менять carrier, сохраняя номер.
- **Universal service fund** — налог на междугородную связь, субсидирует rural areas.

## Пример
**Звонок Бостон → Сан-Франциско (1985):**
1. Местная пара → LEC (Bell Atlantic) в Бостонской LATA.
2. LEC → POP IXC (например, AT&T или MCI или Sprint) в Бостоне.
3. IXC по магистрали → POP в SF LATA.
4. POP → LEC (Pacific Telesis) → SF subscriber.

Three carriers, three bills (хотя customer видит только один).

## Telecommunications Act of 1996

- Открытие local рынка для конкуренции.
- **CLEC** (Competitive Local Exchange Carrier) могут арендовать last-mile у incumbent (ILEC = RBOC).
- Слияние LEC и IXC снова разрешено → возвращение vertical integration.
- В 2010s + интернет ушёл к broadband, регулирование частично потеряло актуальность.

## Связи
- **Базируется на:** [[PSTN — телефонная сеть]] (контекст).
- **Используется в:** анализ архитектуры US-телекома, BGP-peering disputes (legacy of MFJ).
- **Соседи по уровню:** [[Стандартизация сетей]] (regulatory bodies).
- **Противопоставляется:** Европейская модель — обычно one national PTT (Post Telephone Telegraph), потом privatized; разные исторические трэйекtории.

## Подводные камни
- В **России** не было MFJ — Ростелеком унаследовал советскую monopoly, постепенно деregulated.
- Современный VoIP/SIP-trunk обходит традиционные LEC/IXC — экономика 2010s+ сильно изменилась.
- **POP** в IP-эпоху превратился в IXP — но термин и роль всё ещё узнаваемы.

## Дальше читать
- [[PSTN — телефонная сеть]] — техническая часть.
- [[IXP и пиринг]] — современный аналог.
- Tanenbaum, гл. 2, §2.10.3 (стр. PDF 231–234).
