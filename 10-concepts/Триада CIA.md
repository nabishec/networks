---
title: Триада CIA
aliases: ["CIA triad", "Confidentiality", "Integrity", "Availability"]
type: concept
layer: security
chapter: 8
difficulty: basic
prerequisites: []
related: ["[[Безопасность сетей — обзор]]", "[[Виды атак]]"]
tags: [networking, ch08]
---
# Триада CIA — Confidentiality / Integrity / Availability

## TL;DR
Базовая модель целей безопасности (RFC 4949):
- **Confidentiality** — данные читает только авторизованный.
- **Integrity** — данные не изменены в пути или хранении.
- **Availability** — сервис доступен авторизованным пользователям, когда нужно.

Каждая угроза/защита анализируется через эту тройку. Иногда расширяют до **CIAA** (+Authenticity) или **CIANR** (+Non-Repudiation).

## Какую проблему решает
Без формальной модели целей — каждая дискуссия про «безопасность» разъедется в шум. CIA — общий язык: «эта атака на confidentiality»; «defence добавляет integrity, но не availability». Помогает структурировать threat-модели и архитектуру защиты.

## Как работает

| Свойство | Что значит | Атаки | Защита |
|---|---|---|---|
| **Confidentiality** | Незаметность для чужих | eavesdropping, MITM, утечка | шифрование (TLS, IPsec, AES) |
| **Integrity** | Точность данных | modification, замена | hash + signature (HMAC, цифровая подпись) |
| **Availability** | Сервис работает | DoS, DDoS | redundancy, anti-DDoS, rate limit, capacity |

**Compromises:**
- **Confidentiality vs Availability:** строгое шифрование с хранением ключа на одном сервере → если сервер падает, никто не достанет данные.
- **Integrity vs Performance:** signature на каждом сообщении — CPU-cost. На gigabit-пути это значимо.
- **Confidentiality vs Audit:** хочется логировать, но логи — потенциальная утечка.

## Пример
- **HTTPS:** confidentiality (TLS encrypt), integrity (TLS MAC), availability (рассчитывается на CDN/redundancy).
- **DDoS-атака на банк:** атакует только **availability** (deny service), не confidentiality.
- **Утечка БД:** атакует confidentiality (украдены данные).
- **Defacement сайта** (изменён HTML): integrity.
- **Ransomware:** все три — данные зашифрованы (≈ unavailability), потенциально утекли (no confidentiality), могут быть повреждены (no integrity).

## Связи
- **Базируется на:** общая модель security.
- **Используется в:** [[Безопасность сетей — обзор]], threat modeling, audit-методологии (NIST, ISO 27001).
- **Соседи по уровню:** **CIANR** — расширения с authenticity и non-repudiation. **AAA** (authentication, authorization, accounting) — оперативная сторона.
- **Противопоставляется:** «безопасность как набор tooling» — без модели целей выбираешь tools слепо.

## Подводные камни
- **Не все три равноважны для всех систем.** SCADA/promyшленные — availability; банк — integrity; разведданные — confidentiality. Приоритет диктует архитектуру.
- **Trade-offs**: усиление одного часто ослабляет другое.
- **CIA не покрывает privacy** напрямую — это смежная категория.

## Дальше читать
- [[Виды атак]] — конкретные угрозы.
- [[Безопасность сетей — обзор]] — общая картина.
- Tanenbaum, гл. 8, §8.1.1 (стр. PDF 817–819).
