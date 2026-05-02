---
title: DNS — приватность и ODNS
aliases: ["DNS privacy", "ODNS", "ODoH", "TRR"]
type: concept
layer: application
chapter: 7
difficulty: intermediate
prerequisites: ["[[DNS]]", "[[DNS over HTTPS / TLS]]"]
related: ["[[Защита персональной информации]]"]
tags: [networking, ch07]
---
# DNS — приватность и ODNS

## TL;DR
DNS-запрос **раскрывает** ваш IP и **что** вы запрашиваете. **DoH/DoT** шифруют до resolver'а — но resolver всё равно видит. **ODNS / ODoH** (Oblivious DNS, 2019) — двухслойное шифрование: первый узел видит ваш IP но не запрос; второй видит запрос но не IP. **TRR** (Trusted Recursive Resolver) — Mozilla/Chrome отдают DNS публичным resolver'ам (Cloudflare, NextDNS) — выводит DNS из-под контроля ISP, но переносит доверие.

## Какую проблему решает
Даже DoH/DoT не решает централизацию: resolver (Google 8.8.8.8, Cloudflare 1.1.1.1) видит **все** ваши DNS-запросы → строит профиль. Если ISP заменён глобальным resolver'ом — это просто новый «глаз». ODNS пытается сделать запрос **anonymous даже к recursive**.

## Как работает

### ODNS (RFC drafts, 2019)
1. Клиент шифрует actual query с public-key authoritative-сервера.
2. Шлёт на **proxy** (random) — тот не может decrypt.
3. Proxy форвардит на authoritative.
4. Authoritative decrypts, отвечает.

Proxy видит IP клиента + encrypted blob (не знает запрос).
Authoritative видит запрос + IP proxy (не знает клиента).

**Ни один не знает (client_IP, query) одновременно.**

### ODoH (Oblivious DoH, RFC 9230)
- Аналогично ODNS, но через DoH-каналы.
- Cloudflare поддерживает.
- Apple Private Relay включает аналогичную идею.

### TRR (Trusted Recursive Resolver)
- **Mozilla Firefox (2019+):** опционально DoH через Cloudflare/NextDNS вместо ОС-DNS.
- **Chrome:** DoH с тем же resolver, что в системе (если поддерживает DoH).
- **iOS 14+:** Private DNS settings.

**Контроверсия:**
- ISP-функции (parental control, malware-filter) ломаются.
- Корп. сети теряют split-DNS.
- Trust перенесён к глобальному resolver'у (Cloudflare видит всё).

## Пример
**Apple Private Relay (iCloud+):**
- Использует ODNS-подобную архитектуру.
- Двойной TLS-туннель: первый узел Apple, второй — partner CDN (Akamai, Cloudflare).
- Первый видит ваш IP, второй — destination.
- DNS внутри туннеля — оба не видят полный (client, query).

## Связи
- **Базируется на:** [[DNS]] (privacy-проблема), [[DNS over HTTPS / TLS]] (предшественник).
- **Используется в:** Apple Private Relay, экспериментальные deployments в Cloudflare/Mozilla.
- **Соседи по уровню:** [[Защита персональной информации]] (общая privacy-тема), [[Tor — анонимная связь]] (3-hop как onion).
- **Противопоставляется:** plain DNS — provider видит всё; DoH без oblivious — resolver видит всё.

## Подводные камни
- **Не панацея:** ODNS защищает от passive observers, но не от **корреляции** трафика на следующем шаге (HTTPS connection раскрывает destination).
- **Performance** — двойной hop добавляет latency.
- **ISP-functions** теряются: malware-filter, parental control.
- **Концентрация у Cloudflare** — другая риск-точка.

## См. также (прикладное)
RF-circumvention: в РФ DoH/DoT используются прежде всего для обхода DNS-spoofing'а на уровне оператора, не для приватности от глобального resolver'а.
- [[Encrypted DNS — DoH-DoT]] — какой DoH работает (Yandex), какой блокируется (часть Cloudflare).
- [[DNS-туннелирование]] — другая прикладная сторона DNS: канал передачи payload (не приватность, а transport).
- [[applied-rf-status]] — обзор.

## Дальше читать
- [[DNS over HTTPS / TLS]] — простой первый шаг.
- [[Защита персональной информации]] — общая privacy.
- Tanenbaum, гл. 7, §7.1.7 (стр. PDF 701–704).
