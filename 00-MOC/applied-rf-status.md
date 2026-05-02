---
title: Applied RF — статус техник
type: moc
layer: cross-layer
tags: [networking, moc, rf-circumvention, applied]
---
# Applied RF — статус техник

Прикладной слой: что **работает** в РФ на 2026-05-02 для обхода whitelist'ов и DPI. Источники — 10 статей Habr (см. [[_sources]]).

Сортировка: **working → partial → broken**.

## ✅ Working

| Техника | Risk | Сложность | Suitable for | Заметка |
|---|---|---|---|---|
| VLESS-Reality | medium | средне | большинство РФ-провайдеров (не whitelist-mobile) | [[VLESS-Reality]] |
| XTLS-Vision (flow) | medium | низко | Reality-усилитель | [[XTLS-Vision]] |
| xHTTP (packet-up) | medium | средне | обход session-freezing | [[xHTTP]] |
| vnext-цепочка | **high** | высоко | whitelist-mobile + session-freezing | [[vnext-цепочка]] |
| Yandex API Gateway фронтинг | **high** | высоко | mobile-whitelist | [[Yandex API Gateway фронтинг]] |
| Self-Steal — свой домен | medium | средне | стабильность против active probing | [[Self-Steal — свой домен]] |
| WebRTC-туннель | medium | высоко | full-blacklist last-resort | [[WebRTC-туннель]] |
| Split routing | low | низко | повсеместно | [[Split routing]] |
| TURN-relay | low | средне | компонент WebRTC | [[TURN-relay]] |
| MTProxy+FakeTLS («золотые») | medium | средне | Telegram-only | [[MTProxy и FakeTLS]] |
| AmneziaWG | medium | низко | UDP-mobile (вне whitelist) | [[AmneziaWG]] |
| 3X-UI (single-node panel) | low | низко | личное / 1–100 users | [[3X-UI]] |
| Marzban (multi-node panel) | low | средне | 3+ нод, единая подписка | [[Marzban]] |
| Remnawave (heavy panel) | low | высоко | provider 1000+ users, dynamic chains | [[Remnawave]] |

## ⚠️ Partial (работает, но с условиями)

| Техника | Risk | Условие | Заметка |
|---|---|---|---|
| CDN-фронтинг (Cloudflare) | medium | Cloudflare AS попадает в whitelist; classic domain-fronting сломан с 2018 | [[CDN-фронтинг]] |
| Shadowsocks-2022 | medium | без TLS-обёртки → active probing ловит; в whitelist-mode не работает | [[Shadowsocks-2022]] |
| QUIC и mKCP (Hysteria) | medium | UDP/443 нужен whitelist; на mobile часто throttled | [[QUIC и mKCP]] |
| Hysteria-2 | medium | QUIC/UDP-443; нужен whitelist-AS или non-mobile сеть; Salamander обязателен | [[Hysteria-2]] |
| mKCP (Xray transport) | medium | UDP к чужому IP в whitelist-режиме блокируется; в новых Xray есть регрессии | [[mKCP]] |
| SSH-туннелирование | low | low-bandwidth, не для bulk; whitelist-IP может банить | [[SSH-туннелирование]] |
| PingTunnel (ICMP) | low | ~10 KB/s; mobile ICMP часто rate-limit | [[PingTunnel — ICMP]] |
| DNS-туннелирование | low | ~1 KB/s; whitelist-DNS может ограничивать authoritative | [[DNS-туннелирование]] |
| Encrypted DNS (DoH/DoT) | low | Cloudflare DoH часто блокируется; Yandex DoH разрешён | [[Encrypted DNS — DoH-DoT]] |
| DoH в РФ (DNS over HTTPS) | low | Cloudflare/Google часто блокированы; Yandex DoH работает | [[DoH в РФ]] |
| MTProxy+FakeTLS (обычный) | medium | post-handshake-detection ловит; «золотые прокси» дольше живут | [[MTProxy и FakeTLS]] |

## ❌ Broken (на 2026-05-02)

| Техника | Почему сломано | Заметка |
|---|---|---|
| ECH / ESNI | ТСПУ детектирует ECH-extension и дропает соединение (явно в src-01) | [[ECH и ESNI]] |
| Classic domain fronting (cross-domain) | Cloudflare/Google/Amazon закрыли с 2018 | (упоминание в [[CDN-фронтинг]]) |
| OpenVPN без обфускации | DPI ловит handshake-pattern | (вне scope, не делал отдельную) |
| WireGuard (vanilla) | Detection через pattern handshake | (см. [[AmneziaVPN]] для AmneziaWG-flavor) |
| DoT в РФ (DNS over TLS, port 853) | TCP/853 светится; на мобильных РФ дропается на L4 | [[DoT в РФ]] |

## Playbooks (см. [[applied-rf-playbooks]])

- [[PB1 — Yandex API Gateway фронтинг]] — для mobile-whitelist.
- [[PB2 — vnext-цепочка через РФ-мост]] — стабильно при session-freezing.
- [[PB3 — 4-уровневая архитектура за 265₽]] — fault-tolerant.
- [[PB4 — диагностика whitelist]] — диагностика.
- [[PB5 — РФ-каскад с xHTTP+packet-up]] — устойчиво в whitelist-режиме.
- [[PB6 — Nginx+LE с разделением IP]] — anti-IP-profile.
- [[PB7 — basic VLESS-Reality с нуля]] — отправная точка.
- [[PB8 — MTProxy + FakeTLS]] — Telegram-only.
- [[PB9 — Hysteria-2 setup]] — QUIC-VPN с маскировкой под HTTP/3.
- [[PB10 — DNSTT-туннель]] — DNS-tunnel last-resort.
- [[PB11 — Self-Steal-only без РФ-моста]] — single VPS Self-Steal без cascade.

## Tools

**Engines / клиенты:**
- [[Xray-core]] — главный proxy-engine.
- [[Sing-box]] — альтернативный engine.
- [[Hiddify]] — кросс-платформенный клиент.
- [[AmneziaVPN]] — упрощённое self-hosted-решение.

**Веб-панели управления:**
- [[3X-UI и Marzban]] — сводка по web-панелям; атомарно: [[3X-UI]], [[Marzban]], [[Remnawave]].

**MTProxy (Telegram-only):**
- [[mtg]] — Go-реализация MTProxy с FakeTLS.
- [[telemt]] — Rust-реализация MTProxy с transparent splice-fallback.

**Дополнительные транспорты / каналы:**
- [[Cloak]] — pluggable transport для OpenVPN/SS.
- [[coturn]] — TURN-сервер reference-impl (для WebRTC-туннеля).
- [[Cloudflare WARP]] — public WireGuard от Cloudflare.

**Диагностика и мониторинг:**
- [[RealiTLScanner]] — сканер target-сайтов для Reality.
- [[Loyalsoldier geo-files]] — `geosite.dat` / `geoip.dat` для split-routing.
- [[hyperion-cs DPI-checker]] — браузерные/desktop-чекеры DPI-эффектов.
- [[dpi-detector]] — open-source CLI-чекер (Runnin4ik).
- [[cheburcheck.ru]] — публичный web-whitelist-checker.
- [[PingZen]] — SaaS-мониторинг MTProxy.
- [[Zapret GUI]] — локальный DPI-bypass через NFQUEUE.

## Concepts (теоретическая база DPI)

- [[ТСПУ]] — государственный DPI-stack.
- [[Белые списки]] — whitelist-режим.
- [[AS-level whitelist]] — детальнее про блокировку по AS.
- [[Session freezing]] — замораживание после 16 KB.
- [[SNI-фильтрация]] — L7-блокировка по домену.
- [[Active probing]] — DPI-зондирование.
- [[uTLS]] — TLS-fingerprint mimicry.
- [[JA3-JA4 fingerprinting]] — детальный обзор fingerprint-методов.
- [[Behavioral detection]] — ML-классификатор post-handshake (точность 95-99%).
- [[TTL-анализ]] — определение позиции ТСПУ через hop-count.
- [[DPI-фильтрация в РФ]] — сводный обзор всех методов.

## Куда дальше
- [[applied-rf-status]] — этот файл.
- [[applied-rf-playbooks]] — пошаговые сценарии.
- [[applied-rf-glossary]] — короткие определения.
- [[index]] — общий vault-индекс.
