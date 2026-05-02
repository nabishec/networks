---
title: DoH в РФ
aliases: ["DoH в обходе", "DNS over HTTPS в РФ", "DoH"]
type: technique
layer: application
status: partial
status_as_of: 2026-05-02
risk: low
prerequisites: ["[[DNS — приватность и ODNS]]", "[[HTTPS]]"]
related: ["[[Encrypted DNS — DoH-DoT]]", "[[DoT в РФ]]", "[[ECH и ESNI]]"]
sources: [src-05, src-06, src-01]
tags: [networking, rf-circumvention, technique]
---
# DoH в РФ — DNS over HTTPS

## TL;DR
**DoH** (RFC 8484) шифрует DNS-запросы внутри **HTTPS на порту 443**. Внешне неотличим от обычного HTTPS-трафика → DPI не может тривиально срезать по порту/протоколу. В РФ-2026: **Cloudflare DoH (1.1.1.1) и Google DoH (8.8.8.8) часто блокированы** на L3/L4 (особенно у мобильных операторов в whitelist-режиме); **Yandex DoH (77.88.8.8)** — в большинстве сетей разрешён, попадает в whitelist по AS Яндекса.

## Какую проблему решает
- Обычный DNS (port 53) идёт **plain-text** → провайдер видит запрашиваемые домены и может **подменять** ответы (DNS-spoofing для блокировок) или просто логировать.
- DoH прячет запросы внутрь HTTPS → провайдер видит лишь TLS-handshake к public-resolver'у.
- Снимает **DNS-step** в SNI-фильтрации: до DoH провайдер мог по DNS-запросу понять, к какому домену клиент идёт, ещё до TLS-handshake.

## Как работает
1. Клиент шлёт `POST /dns-query` (или `GET ?dns=...`) на HTTPS-resolver — например, `https://cloudflare-dns.com/dns-query`, `https://dns.google/dns-query`, `https://common.dns.yandex.net/dns-query`.
2. Тело запроса — wire-format DNS-сообщение или JSON.
3. Ответ — DNS-response внутри HTTPS-body.
4. Внешне = обычный HTTPS-traffic → не отличается от `google.com` визитов.

## Где ломается / почему может не работать
- **L3/L4-блокировка IP-адресов resolver'ов:** мобильные операторы РФ дропают пакеты к `1.1.1.1`, `8.8.8.8` уже на маршрутизации (src-05). Yandex DNS пропускают, т.к. AS 13238 в whitelist.
- **DoH-fingerprint:** TLS-handshake к 1.1.1.1 имеет узнаваемый pattern (cert SNI = `cloudflare-dns.com`) → DPI может детектить через L7 даже когда IP пропущен.
- **Не решает SNI-фильтрацию самого приложения:** DoH прячет только DNS-step. Когда браузер потом откроет TLS к заблокированному домену, DPI всё равно видит SNI.
- **iOS/Android default DoH** (Private DNS) часто настроен на Cloudflare → в РФ откатывается на plain-DNS → утечка.
- **РКН-рекомендации 2024:** прямо просят владельцев ресурсов **не использовать** Cloudflare-сервисы (включая 1.1.1.1) → блокировки усиливаются.

## Минимальный пошаговый сценарий

**Linux (systemd-resolved):**
```ini
# /etc/systemd/resolved.conf
[Resolve]
DNS=77.88.8.8#common.dns.yandex.net
DNSOverTLS=opportunistic
```
*(Yandex поддерживает и DoH, и DoT.)*

**Firefox:**
`about:preferences#privacy` → DNS over HTTPS → Custom: `https://common.dns.yandex.net/dns-query`.

**Свой DoH-resolver на VPS** (вне whitelist-блокировок):
- AdGuardHome / dnsdist / Pi-hole с DoH-listener на 443.
- Получаешь зашифрованный канал, который нельзя отличить от HTTPS.

**Проверка:**
```bash
curl -H 'accept: application/dns-json' \
  'https://common.dns.yandex.net/dns-query?name=habr.com&type=A'
```

## Что нужно
- Браузер/ОС с DoH-поддержкой (современные).
- DoH-resolver, который реально доступен в РФ-сети (Yandex, NextDNS-anycast, свой VPS).

## Связи
- **Базируется на:** [[DNS — приватность и ODNS]], [[HTTPS]], [[TLS — рукопожатие]].
- **Парный с:** [[DoT в РФ]] — тот же замысел, другой транспорт. DoH в РФ работает чаще, чем DoT (см. [[DoT в РФ]]).
- **Соседи:** [[ECH и ESNI]] (защищает SNI после DNS-step), [[DNS-туннелирование]] (использует DoH как несущую).
- **Сравнение в:** [[Encrypted DNS — DoH-DoT]].

## Источники
- src-05 — эмпирически: 8.8.8.8 дропается на мобильных РФ, Yandex DNS пропускается.
- src-06 — Encrypted DNS как часть каскада обхода.
- src-01 — упоминание DoH/DoT как один из 6 способов; помечает ECH как сломанный.
- [Минимизация рисков использования DNS-over-TLS (DoT) и DNS-over-HTTPS (DoH) — habr 523676](https://habr.com/ru/articles/523676/)
- [Кабысдох — DoH-припарка от русского firewall — habr 538806](https://habr.com/ru/articles/538806/)
- [Приватный DNS: защищаемся от слежки провайдера — habr 945660](https://habr.com/ru/articles/945660/)
