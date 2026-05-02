---
title: DNS spoofing
aliases: ["DNS cache poisoning", "DNSSEC", "Отравление DNS-кэша"]
type: concept
layer: security
chapter: 8
difficulty: intermediate
prerequisites: ["[[DNS]]"]
related: ["[[Виды атак]]", "[[ARP spoofing]]"]
tags: [networking, ch08]
---
# DNS spoofing (cache poisoning)

## TL;DR
Атакующий внедряет в DNS-кэш recursive resolver'а **поддельный ответ** для популярного домена → жертвы получают неправильный IP → MITM. Главная защита — **DNSSEC** (RFC 4033+) — криптографические подписи DNS-ответов. Также **DoH/DoT** скрывают DNS от подмены на пути.

## Какую проблему решает
Понимать, как через **DNS** можно перенаправить пользователей на фишинговые сайты, и почему DNSSEC/DoH важны.

## Как работает

**Классические сценарии:**

### 1. Off-path DNS spoofing (Kaminsky 2008)
- Атакующий не на пути, но шлёт DNS-resolver массу запросов.
- Параллельно шлёт **подложные ответы** с угаданными txid и port.
- Если угадал до прихода реального ответа от auth-сервера → cache poisoned.
- **Mitigation:** случайный source port (делает angle угадать в 2³² раз больше).

### 2. On-path attack
- Атакующий на пути (ISP, Wi-Fi, корпоративная сеть).
- Видит DNS-запросы → шлёт фальшивый ответ быстрее authoritative.
- **Mitigation:** DoT/DoH — encrypted; но если корпоративный TLS-MITM — обходится.

### 3. Authoritative compromise
- Атакующий контролирует authoritative server какого-то домена → отвечает что хочет.
- **Mitigation:** DNSSEC сделает unsigned ответ детектируемым.

### DNSSEC
- Каждая RRset подписана **RRSIG** ключом (DNSKEY).
- DNSKEY подписан DS-записью у parent-зоны.
- Цепочка: root → TLD → domain.
- Resolver валидирует от root.
- **NSEC/NSEC3** — proof of non-existence (нет записи).

**Развёртывание:**
- Root, .com, .org, .ru (и др.) подписаны.
- Конкретные домены — на 30-50% подписаны (растёт).
- Validating resolvers — Google 8.8.8.8, Cloudflare 1.1.1.1 — да.

## Пример
**Phishing атака на банк:**
1. Атакующий поднимает фальшивый сайт `mybank.com`-копию.
2. Spoof'ит DNS — `mybank.com → his_IP`.
3. Жертва вводит mybank.com → попадает на фейк → вводит логин/пароль.
4. **HTTPS не помогает**, если жертва игнорирует cert-warning. Современные браузеры жёстко предупреждают.

**Защита:** DNSSEC + HSTS + Certificate Transparency + EV-сертификаты.

## Связи
- **Базируется на:** [[DNS]] (атака на него).
- **Используется в:** [[Виды атак]] (категория spoofing).
- **Соседи по уровню:** [[ARP spoofing]] — соседняя по слою.
- **Противопоставляется:** DNSSEC, DoH/DoT — защиты.

## Подводные камни
- **DNSSEC сложен** — много ключей, ротации, нюансы. Mis-configuration → выпадение домена из интернета.
- **DoH/DoT** скрывают DNS, но **provider DoH** (Cloudflare, Google) видит всё → перенесение доверия.
- **DNSSEC ≠ encryption** — ответ всё ещё читается посередине, просто не подделать.

## Дальше читать
- [[DNS]] — что атакуется.
- [[Виды атак]] — общая категория.
- Tanenbaum, гл. 8, §8.12.2 (стр. PDF 928–931).
