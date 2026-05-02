---
title: Applied RF — глоссарий
type: moc
layer: cross-layer
tags: [networking, moc, rf-circumvention, glossary]
---
# Applied RF — глоссарий

Короткие определения с указателем на детальные заметки.

## Регулятор и инфраструктура

- **ТСПУ** ([[ТСПУ]]) — Технические средства противодействия угрозам. DPI-stack ФЗ-90 «суверенный интернет», управляемый РКН через ЦМУ ССОП. Производитель — РДП.РУ. Размещены на узлах операторов.
- **РКН** — Роскомнадзор. Регулятор связи и интернета в РФ.
- **ЦМУ ССОП** — Центр мониторинга и управления сетью связи общего пользования.
- **CGNAT** ([[NAT]]) — Carrier-Grade NAT. Узел `198.18.x.x` — частая маркер ТСПУ-карты в мобильных сетях.

## Стратегии фильтрации

- **DPI** ([[Брандмауэр]] / [[DPI-фильтрация в РФ]]) — Deep Packet Inspection.
- **Whitelist (Default-Deny)** ([[Белые списки]]) — «всё запрещено по умолчанию», пропускаются только разрешённые. Mobile в РФ-2026: ~0.14% IP.
- **Blacklist** — «всё разрешено по умолчанию», блокируются конкретные. Эпоха до 2025.
- **AS-level whitelist** ([[AS-level whitelist]]) — блокировка/разрешение по ASN. 72 AS allowed; 391 пострадало коллатерально (src-09).
- **SNI-фильтрация** ([[SNI-фильтрация]]) — чтение SNI-extension в TLS-handshake → блокировка по домену.
- **Session freezing** ([[Session freezing]]) — DPI замораживает сессию после 16 KB без RST.
- **Active probing** ([[Active probing]]) — DPI сама подключается к suspicious IP, проверяет ответ.
- **TLS fingerprint** ([[uTLS]]) — JA3/JA4-классификация по handshake-параметрам.
- **JA3 / JA4 fingerprinting** ([[JA3-JA4 fingerprinting]]) — детальный обзор обоих методов; что обходит [[uTLS]], а что — нет.
- **Behavioral detection (post-handshake)** ([[Behavioral detection]]) — ML-классификатор VPN/прокси по поведению трафика после handshake; точность 95-99% для незамаскированных протоколов.
- **TTL-анализ** ([[TTL-анализ]]) — определение позиции ТСПУ через hop-count входящих RST.

## Туннели и обёртки

- **VLESS** — protocol-родитель в Xray. Stateless, без шифрования (защищается на TLS-level).
- **Reality (XTLS-Reality)** ([[VLESS-Reality]]) — VLESS+TLS, маскирующийся под чужой target-сайт.
- **XTLS-Vision** ([[XTLS-Vision]]) — flow для VLESS, убирает TLS-in-TLS-аномалию.
- **xHTTP packet-up** ([[xHTTP]]) — обёртка VLESS в HTTP/2-POST'ы; обходит [[Session freezing]].
- **vnext** ([[vnext-цепочка]]) — chain между Xray-нодами.
- **MTProxy** ([[MTProxy и FakeTLS]]) — proxy-протокол Telegram. Эволюция: Obfuscated2 (DD) → FakeTLS (EE).
- **FakeTLS** ([[MTProxy и FakeTLS]]) — маскирует первые байты как TLS 1.3 ClientHello + HMAC-SHA256.
- **Shadowsocks-2022** ([[Shadowsocks-2022]]) — обновлённый SS со случайными paddings и AEAD.
- **Hysteria / Tuic** ([[QUIC и mKCP]]) — QUIC-based VPN с custom congestion control.
- **Hysteria-2** ([[Hysteria-2]]) — QUIC/UDP-443 + Brutal-congestion + Salamander-обфускация. Маскируется под HTTP/3.
- **mKCP** ([[mKCP]]) — Xray transport (reliable-UDP/KCP) с маскировкой UDP-header под BitTorrent/SRTP/WeChat.
- **AmneziaWG** ([[AmneziaWG]]) — обфусцированный WireGuard от команды Amnezia (junk-packets, замена magic-byte). Vanilla WireGuard в РФ-2025 — broken.

## Маскировка

- **Self-Steal** ([[Self-Steal — свой домен]]) — свой домен с реальным сайтом, прокси на specific path.
- **CDN-фронтинг** ([[CDN-фронтинг]]) — backend за CDN (Cloudflare, CloudFront).
- **Yandex API Gateway фронтинг** ([[Yandex API Gateway фронтинг]]) — РФ-облачный фронт.
- **uTLS** ([[uTLS]]) — TLS-handshake mimicry под Chrome/Firefox.
- **«Золотые прокси»** — MTProxy с маскировкой под trusted-домены (yandex.ru, gosuslugi.ru, sberbank.ru).

## Каналы last-resort

- **WebRTC-туннель** ([[WebRTC-туннель]]) — DTLS-SRTP канал поверх UDP, как видеозвонок.
- **TURN-relay** ([[TURN-relay]]) — relay для WebRTC (coturn).
- **DNS-туннелирование** ([[DNS-туннелирование]]) — payload в DNS-запросах (xDNS, DNSTT). ~1 KB/s.
- **PingTunnel** ([[PingTunnel — ICMP]]) — payload в ICMP Echo. ~10 KB/s.
- **SSH-туннелирование** ([[SSH-туннелирование]]) — `ssh -D` SOCKS5-tunnel.

## Ecosystem Cloud

- **Yandex Cloud** (AS 13238) — российский cloud, обычно в whitelist mobile-операторов.
- **VK Cloud** — аналогично.
- **DiNet** / **EDGE** — региональные дешёвые VPS-провайдеры в whitelist-AS.
- **Cloudflare** (AS 13335) — глобальный CDN, периодически попадает в коллатеральный blacklist.

## Инструменты

- **Xray-core** ([[Xray-core]]) — proxy engine #1 (форк V2Ray).
- **Sing-box** ([[Sing-box]]) — альтернативный engine, чище код, sing-routing.
- **Hiddify** ([[Hiddify]]) — клиент cross-platform на sing-box.
- **3X-UI / Marzban / Remnawave** ([[3X-UI и Marzban]]) — сводное сравнение веб-панелей.
- **3X-UI** ([[3X-UI]]) — простая single-node панель для Xray; bash-инсталлятор; ниша — личное/team.
- **Marzban** ([[Marzban]]) — Python-панель Xray с master+nodes-архитектурой и mTLS между ними.
- **Remnawave** ([[Remnawave]]) — Node.js-панель Xray с Subscribe-ID, dynamic chains, Python-SDK; для крупных провайдеров.
- **AmneziaVPN / AmneziaWG** ([[AmneziaVPN]]) — упрощённое self-hosted-решение.
- **mtg / telemt / MTProxy** ([[MTProxy и FakeTLS]]) — реализации MTProxy.
- **mtg** ([[mtg]]) — Go-реализация MTProxy с встроенным FakeTLS (`9seconds/mtg`); де-факто стандарт 2026 г.
- **telemt** ([[telemt]]) — Rust-реализация MTProxy с transparent TCP-splice fallback на реальный backend при probing.
- **Zapret / Zapret GUI** ([[Zapret GUI]]) — локальный DPI-bypass через NFQUEUE (фрагментация TCP, fake-headers); работает только против non-whitelist DPI.

## Encrypted DNS / приватность

- **DoH** ([[DoH в РФ]]) — DNS over HTTPS (RFC 8484, port 443). Сводка — [[Encrypted DNS — DoH-DoT]].
- **DoT** ([[DoT в РФ]]) — DNS over TLS (RFC 7858, port 853). На мобильных РФ почти всегда заблокирован.
- **ECH / ESNI** ([[ECH и ESNI]]) — Encrypted ClientHello. **Сломано** в РФ.
- **ODNS / ODoH** ([[DNS — приватность и ODNS]]) — Oblivious DNS.

## Метрики и инструменты исследования

- **JA3 / JA4** ([[JA3-JA4 fingerprinting]]) — fingerprint TLS-handshake (cipher-suites + extensions).
- **PingZen** ([[PingZen]]) — SaaS-мониторинг с уникальным MTProxy-checker'ом (полный handshake-путь Telegram-клиента).
- **cheburcheck.ru** ([[cheburcheck.ru]]) — публичный web-whitelist-checker с базой знаний.
- **hyperion-cs DPI-checker** ([[hyperion-cs DPI-checker]]) — набор браузерных и desktop-чекеров для 16-20 KB session-freezing, AS-whitelist, SNI-фильтрации.
- **dpi-detector** ([[dpi-detector]]) — open-source CLI-tool для самодиагностики (Runnin4ik).
- **RealiTLScanner** ([[RealiTLScanner]]) — сканер пригодных target-сайтов для VLESS-Reality.
- **Loyalsoldier geo-files** ([[Loyalsoldier geo-files]]) — `geosite.dat` / `geoip.dat` для split-routing.

## Куда дальше
- [[applied-rf-status]] — что работает.
- [[applied-rf-playbooks]] — пошаговые сценарии.
- [[index]] — общий vault.
