---
title: RealiTLScanner
aliases: ["RealiTLS scanner"]
type: tool
layer: application
status: working
status_as_of: 2026-05-02
risk: low
prerequisites: ["[[TLS — рукопожатие]]"]
related: ["[[VLESS-Reality]]", "[[Active probing]]"]
sources: [src-03]
tags: [networking, rf-circumvention, tool]
---
# RealiTLScanner

## TL;DR
**Сканер пригодных target-сайтов для VLESS-Reality.** На вход — диапазон IP или AS; на выход — список IP/SNI, которые удовлетворяют требованиям Reality:
- TLS 1.3-only (без fallback).
- ALPN включает `h2`.
- Сертификат от **публичного CA** (не self-signed).
- TLS не отвечает на ECH (классический режим).
- Нет **active probing detection** (отвечает корректно даже на «случайные» fake-handshake).

## Какую проблему решает
Подбирать `dest`/`serverNames` для Reality «руками» через `openssl s_client` — медленно. Reality требует, чтобы target держал TLS 1.3 и реальный ServerHello forward'ился клиенту. Не каждый сайт подходит. RealiTLScanner делает массовое сканирование за минуты.

## Как работает
1. Берёт IP-range или ASN-список.
2. Для каждого IP открывает TLS 1.3 ClientHello c популярными SNI.
3. Парсит ServerHello, certificate-chain, ALPN.
4. Фильтрует по критериям Reality.
5. Выводит список «годных» (IP, SNI, fingerprint, cert-issuer, response-time).

## Минимальный пошаговый сценарий
```bash
# Скачать (Go-binary, GitHub: hawshemi/RealiTLScanner)
go install github.com/hawshemi/RealiTLScanner@latest

# Сканирование одного IP-range:
RealiTLScanner -addr 1.1.1.0/24 -port 443

# Сканирование ASN (требует maxmind-geoip):
RealiTLScanner -asn AS15169 -port 443 -out google-targets.txt
```

Вывод — табличка `IP:port  SNI  ALPN  Cert-Issuer  TLS-Version  RTT`.

## Где ломается / лимиты
- Сканирование больших ASN — десятки минут.
- Some провайдеры rate-limit'ят TLS-handshake-burst → false-negative.
- Не проверяет **stability over time** — target может «выпасть» через неделю.
- Не учитывает РФ-специфику: target должен быть достижим **из РФ-VPS** (если строится cascade).

## Связи
- **Используется для:** настройки [[VLESS-Reality]] — выбор `dest`/`serverNames`.
- **Соседи по уровню:** [[Active probing]] (DPI-сторона того же сканирования).
- **Аналоги:** ручная проверка через `openssl s_client -connect target:443 -tls1_3 -alpn h2`.

## Источники
- src-03.
