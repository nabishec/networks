---
title: VPN
aliases: ["Virtual Private Network", "Site-to-site VPN", "Remote access VPN"]
type: concept
layer: security
chapter: 8
difficulty: basic
prerequisites: ["[[Туннелирование]]"]
related: ["[[IPsec]]", "[[TLS — рукопожатие]]", "[[Защита персональной информации]]"]
tags: [networking, ch08]
---
# VPN — Virtual Private Network

## TL;DR
**Логически приватная сеть** через **публичную** инфраструктуру. Туннель шифрует и аутентифицирует трафик между endpoints. Два главных типа: **site-to-site** (между офисами компании, постоянный туннель) и **remote access** (ноутбук сотрудника → корпоративный шлюз). Технологии: [[IPsec]], **OpenVPN** (TLS-based), **WireGuard** (modern, simple, ChaCha20). Также — **commercial consumer VPN** (NordVPN, Mullvad) для приватности/обхода блокировок.

## Какую проблему решает
- **Корп.:** соединить географически разнесённые офисы и удалённых сотрудников **как один LAN**, поверх дешёвого интернета вместо дорогих leased lines.
- **Personal:** скрыть свой IP от сайтов, обойти географические ограничения, защитить трафик в публичном Wi-Fi.
- **Censorship circumvention:** обход блокировок (через VPN-сервер в другой стране).

## Как работает

**Туннелирование** ([[Туннелирование]]):
- Trafic encapsulated в encrypted tunnel.
- Снаружи виден только обмен между endpoints туннеля.

**Архитектура site-to-site:**
```
Office Moscow                   Office SPb
[10.1.0.0/16] -- GW (1.1.1.1) === Internet === GW (2.2.2.2) -- [10.2.0.0/16]
                          ←  IPsec/WireGuard  →
```

**Remote access:**
```
Employee laptop -- Internet -- VPN GW (corp.com:443) -- Corporate LAN
```

**Технологии:**

| | IPsec | OpenVPN | WireGuard | TLS-based proprietary |
|---|---|---|---|---|
| Уровень | L3 | L3-L4 | L3 | L7 |
| Транспорт | IP/ESP, UDP NAT-T | TCP/UDP 1194 | UDP | TCP/443 |
| Crypto | AES, ChaCha20 | TLS suite | ChaCha20-Poly1305 | TLS |
| Сложность | сложно | средне | просто | varies |
| Performance | hardware-accelerated | software | very fast | depends |

**Modern:** WireGuard вытесняет, особенно для personal-VPN.

## Пример
- **Corporate VPN:** Cisco AnyConnect (TLS-based, на 443) или WireGuard. Сотрудник дома → шлюз → доступ к внутренним ресурсам.
- **NordVPN/Mullvad/ProtonVPN:** consumer VPN. Маршрутизирует **весь** трафик через шифрованный канал к серверу в выбранной стране.
- **Tor** — НЕ VPN: 3 hops onion routing для anonymity, не privacy от VPN-провайдера.

## Связи
- **Базируется на:** [[Туннелирование]] (механизм), [[IPsec]] / TLS / WireGuard (конкретные).
- **Используется в:** corporate networks, personal privacy, censorship circumvention.
- **Соседи по уровню:** **SD-WAN** — программно-управляемые VPN-overlay'и; **Zero Trust Network Access** — современный paradigm («не VPN», а per-app proxying).
- **Противопоставляется:** unencrypted internet — приватность нулевая.

## Подводные камни
- **VPN-провайдер = новый «подслушиватель»:** видит всё, что видел ISP до VPN. Доверие просто переносится.
- **VPN ≠ Tor.** VPN — один hop. Tor — 3 hops с onion routing. Anonymity модели разные.
- **DNS leak:** VPN обходится по network-traffic, но system DNS может идти к ISP-resolver'у → leak. Нужно DNS через VPN.
- **WebRTC leak:** в браузере можно узнать local IP через STUN — обходит VPN. Disable WebRTC или firewall.
- **Kill switch:** если VPN падает — не fallback'ить на open internet (для critical privacy).

## См. также (прикладное)
RF-circumvention: censorship-circumvention VPN в РФ — самостоятельный пласт техник, отличающихся от классических VPN маскировкой и устойчивостью к DPI.
- [[applied-rf-status]] — таблица: что работает на 2026-05-02.
- [[VLESS-Reality]] — основная censorship-resistant техника.
- [[AmneziaVPN]] — упрощённое self-hosted-решение с обфускацией.
- [[Xray-core]] / [[Sing-box]] — engine'ы.
- [[ТСПУ]] / [[DPI-фильтрация в РФ]] — что приходится обходить.
- [[Белые списки]] / [[Session freezing]] — главные «убийцы» обычных VPN.

## Дальше читать
- [[IPsec]] — main технология.
- [[Туннелирование]] — теория.
- [[Tor — анонимная связь]] — другая модель.
- Tanenbaum, гл. 8, §8.10.2 (стр. PDF 915–917).
