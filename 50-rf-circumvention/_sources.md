---
title: Источники — статьи про обход блокировок РФ
type: meta
tags: [meta, sources, rf-circumvention]
---
# Источники

Загружено через WebFetch 2026-05-02. **TL;DR в одну строку** + детали внутри.

## Таблица

| id | URL | Заголовок | Автор | Опубл. | Прочит. | TL;DR |
|---|---|---|---|---|---|---|
| **src-01** | [1027276](https://habr.com/ru/articles/1027276/) | «Это — всё что вам надо знать о белых списках: как устроены и 6 способов обхода» | zarazaex | 2026-04-25 | 2026-05-02 | Двухуровневая фильтрация мобильных операторов (L3 IP + L7 SNI), пропускают ~0.14% адресов; обходят через российские VPS / Yandex Cloud Functions / API Gateway. |
| **src-02** | [985674](https://habr.com/ru/articles/985674/) | «Гайд по обходу "белых списков" и настройке цепочки рабочие варианты, почему ваш VPN может не работать» | adlayers | 2025-01-15 | 2026-05-02 | DPI замораживает длинные TCP-сессии (не RST); решение — двухуровневая цепочка: РФ-нода (мост) + зарубежная (выход) на VLESS-Reality + xHTTP. |
| **src-03** | [1021160](https://habr.com/ru/articles/1021160/) | «Мой VPN пережил белые списки. Архитектура из 4 уровней за 265₽ в месяц» | Sergei_creator | 2026-04-09 | 2026-05-02 | 4-уровневая архитектура с резервами: VLESS-Reality + CDN-фронтинг + Cloudflare + relay через РФ-облако + WebRTC, ~265₽/мес. |
| **src-04** | [1008164](https://habr.com/ru/articles/1008164/) | «Белые списки добрались до Москвы: изучаем механику "отсечки" в 16 килобайт» | Anton19891 | 2026-03-09 | 2026-05-02 | Эмпирическое тестирование 1 млн доменов из Tranco; механика «отсечки» 16-20 КБ; ASN не обязателен (разрешённый домен может «протащить» к чужим серверам); SNI-mask на поддомены работает. Замена 988862 (403). |
| **src-05** | [990190](https://habr.com/ru/articles/990190/) | «Reality in Whitelists» | habrconnect (перевод Андрея Amonoc) | 2026-02-17 | 2026-05-02 | Эмпирическое исследование: РФ-мобильный — whitelist-модель; работают только разрешённые сервисы, остальное не доходит. |
| **src-06** | [979128](https://habr.com/ru/articles/979128/) | «Эпоха "белых списков": почему ваши конфиги в декабре 2025 года начали превращаться в тыкву, и что нас ждет…» | adlayers (Иван) | 2025-12-21 | 2026-05-02 | Default-deny у провайдеров; решение — каскад через РФ-сервера, xHTTP packet-up, Self-Steal-домен, нестандартные порты, ECH. |
| **src-07** | [1013122](https://habr.com/ru/articles/1013122/) | «Белые списки на домашнем интернете — уже скоро и как подготовиться?» | Soldier22 | 2026-03-21 | 2026-05-02 | Гонка с провайдерами: разделять входящий/исходящий через разные IP, TURN-серверы, WebRTC-туннели, маскировка под HTTPS, Nginx + Let's Encrypt. |
| **src-08** | [799751](https://habr.com/ru/articles/799751/) | «Надежный обход блокировок в 2024: протоколы, клиенты и настройка сервера от простого к сложному» | Deleted-user | 2024-03-14 | 2026-05-02 | Обзор протоколов на 2024: VLESS+XTLS-Reality (рекомендуемое), Shadowsocks-2022, SSH, QUIC/mKCP, CDN-фронтинг (Cloudflare/CloudFront), DNSTT, PingTunnel. |
| **src-09** | [997088](https://habr.com/ru/articles/997088/) | «РКН создали белый список для 72 AS, но пострадали 391 AS (>225 млн IP адресов)» | 0ka | 2025-02-16 | 2026-05-02 | Selektiwный whitelist DPI на ASN-уровне для 72 AS; коллатерально пострадали 391 AS (Cloudflare/OVH/Hetzner/DigitalOcean), >225 млн IP. |
| **src-10** | [1018740](https://habr.com/ru/articles/1018740/) | «Не соглашаясь с Большим Братом. Telegram и MTProxy» | Ilya519 | 2026-04-03 | 2026-05-02 | MTProxy технически: Obfuscated2 (DD), FakeTLS (EE), HMAC-SHA256 hidden-handshake; «золотые прокси» (маскировка под Yandex/Госуслуги/СберБанк); мониторинг через PingZen. |

---

## Детали по статьям

### src-01 — Белые списки: 6 способов обхода (zarazaex, 25.04.2026)
- **Техники:** VLESS+Reality, Yandex Cloud Functions, Yandex API Gateway, olcRTC (туннель через видеозвонки), xDNS (туннелирование через DNS), TURN relay, ECH/ESNI помечены **как неработающие**.
- **Инструменты:** Xray, Sing-box, Cloudflare (для DNS-делегирования).
- **Концепты:** DPI, SNI-фильтрация, IP-whitelisting, L3/L7 фильтрация, TLS-fingerprint, TTL-анализ для определения геопозиции ТСПУ.
- **Playbook (Yandex API Gateway):** домен → Cloudflare → Let's Encrypt в Yandex CM → API Gateway с OpenAPI → CNAME → трафик через разрешённый IP Яндекса.

### src-02 — Гайд по цепочке (adlayers, 15.01.2025)
- **Техники:** VLESS+Reality, xHTTP (packet-up), XTLS-RPRX-Vision (flow), vnext (chain между нодами).
- **Инструменты:** Xray-core ≥ v25.12.8, hynet.space, Amnezia, VoxiProxy, MamontVPN, SayVPN, DuckVPN, Remnawave.
- **Концепты:** session freezing вместо RST, NewSessionTicket-fingerprint, geo-файлы для split routing.
- **Playbook:** **2-уровневая цепочка** = РФ-мост (Yandex/VK/EDGE) + зарубежный выход; routing на мосту: RU-домены → DIRECT, остальное → chain.

### src-03 — 4-уровневая архитектура (Sergei_creator, 09.04.2026)
- **Техники:** VLESS-Reality, CDN-фронтинг через Cloudflare, relay через РФ-облако, WebRTC-туннель, split routing, SNI rotation.
- **Инструменты:** Xray-core, 3X-UI, Hiddify, Shadowrocket, RealiTLScanner, OlcRTC.
- **Концепты:** ТСПУ, IP-leak, preemptible VM, SFU-архитектура (WebRTC).
- **Playbook:** заказать VPS в Нидерландах + домен; Claude Code генерирует план/скрипты деплоя; SSH-команды + клиенты установить.

### src-04 — Белые списки в Москве (Anton19891, 09.03.2026)
Замена для исходной 988862, остававшейся 403. Содержание статьи 1008164:
- **Техники:** VLESS+Reality (рекомендуется как «маскируется под обычный трафик»), [[AmneziaWG]] (модифицированный WireGuard), **Zapret GUI** (подбор DPI-стратегий под конкретного оператора), двойной VPS (РФ-мост + зарубежный выход).
- **Концепты:** механика «16-20 КБ отсечки» — DPI прерывает соединение ровно после ~16 КБ ([[Session freezing]]); SNI-mask на поддомены (если `ok.ru` разрешён, пройдут и `*.ok.ru`); ASN-совпадение **желательно, но не обязательно** — некоторые «белые» домены могут «протащить» трафик к чужим AS.
- **Эмпирика:** автор гонял `curl` HTTPS-запросов по 1 млн доменов рейтинга **Tranco**; обнаружены аномалии в `.co.uk`-зоне.
- **Инструменты диагностики:** Tranco top-1M, curl, Zapret.

### src-05 — Reality in Whitelists (habrconnect, 17.02.2026)
- **Техники:** двухуровневая фильтрация (DNS + IP/Transport), блокировка 8.8.8.8 vs Yandex DNS 77.88.8.8, CGNAT через шлюз `198.18.9.129`, IP-фильтрация на оператор-уровне.
- **Инструменты исследования:** Wireshark (PCAPNG), dig, ping, curl, traceroute, tcpdump, mtr.
- **Концепты:** positive-filtering (whitelist), CGNAT, DNS-resolution без гарантии reachability, routing-level packet drops.
- **Playbook (диагностика):** dig через разные DNS → ping ICMP → curl с timeout → traceroute (узел 198.18.x.x = ТСПУ?) → Wireshark `SYN без SYN-ACK`.

### src-06 — Эпоха белых списков (adlayers, 21.12.2025)
- **Техники:** каскад через РФ-сервера, xHTTP+packet-up (фрагментация), Self-Steal (свой домен вместо чужого), encrypted DNS, нестандартные порты (8443, 2053, 30000+), ECH.
- **Инструменты:** Xray-core, Marzban, 3x-ui, Remnawave, Amnezia, Happ, v2raytun, Shadowrocket; hynet.space, VoxiProxy; Yandex/VK Cloud, DiNet.
- **Концепты:** Default-Deny, DPI, SNI-проверка, ТСПУ, TLS 1.3, ECH.
- **Playbook:** РФ-входной сервер с iptables → свой домен → xHTTP+packet-up → forward на зарубежный VPS.

### src-07 — Белые списки на домашнем (Soldier22, 21.03.2026)
- **Техники:** разделение входящего/исходящего через разные IP, TURN-серверы, WebRTC-туннели, обфускация протоколов, маскировка под HTTPS, DTLS-обёртки UDP.
- **Инструменты:** Hynet.space, AmneziaVPN/AmneziaWG, RTC Tunnel, whitelist-bypass-скрипты, coturn, Nginx + Let's Encrypt.
- **Концепты:** «гонка вооружений», профайл IP, маркеры подозрительных узлов, коммерциализация блокировок.
- **Playbook:** «белый» IP → легитимный сайт на Nginx + LE → разделить потоки через разные IP → не часто менять адреса → обфускация.

### src-08 — Надёжный обход 2024 (анонимный, 14.03.2024)
- **Техники:** **VLESS+XTLS-Reality** (рекомендация №1), Shadowsocks-2022, SSH-туннелирование, QUIC/mKCP, HTTP/2 multiplexing, **CDN-фронтинг** (Cloudflare, Amazon CloudFront), Cloudflare WARP, PingTunnel (ICMP), DNSTT (DNS-туннель).
- **Инструменты:** Xray-core, Sing-box, Hiddify-Next, Nekobox/Nekoray, v2rayN/NG, NekoboxForAndroid, Streisand, FoxRay, v2raytun, Amnezia.
- **Концепты:** ТСПУ, TLS fingerprinting, **active probing**, **uTLS** (изменение клиентского fingerprint'а), **XTLS-Vision** (TLS-in-TLS-маскировка), Reality (маскировка под реальный сайт), split tunneling, white/black-листы.
- **Playbook (basic VLESS-Reality):** найти популярный TLS-сайт → curl-проверить → установить XRay → UUID + x25519-ключи → JSON-конфиг → systemd → собрать VLESS-link → клиент (Hiddify/Nekoray) или QR.

### src-09 — РКН и 72 AS (0ka, 16.02.2025; повторно сканировалось 10.02.2026)
- **Техники:** DPI-фильтрация на уровне ASN, белый список доменов, TCP-«замораживание» после 16 КБ.
- **Инструменты:** hyperion-cs DPI-checker (https://hyperion-cs.github.io/dpi-checkers/ru/tcp-16-20/), dpi-detector, cheburcheck.ru.
- **Концепты:** AS, ASN, IP-prefix-space, CDN-провайдеры (Cloudflare, OVH, Hetzner, DigitalOcean), коллатеральный ущерб.
- **Playbook:** аналитика, не пошаговый.

### src-10 — Telegram и MTProxy (Ilya519, 03.04.2026)
- **Техники:** Obfuscated2 (DD-поколение), Fake TLS (EE-поколение), HMAC-SHA256-аутентификация, post-handshake detection, behavioral analysis.
- **Инструменты:** MTProxy (официальный репозиторий Telegram), telemt (Rust), mtg (Go), telegram-proxy-collector, PingZen (мониторинг), ТСПУ (РДП.РУ).
- **Концепты:** «суверенный интернет» (2019 закон), «золотые прокси» (маскировка под Яндекс/Госуслуги/СберБанк), гонка вооружений, ML-detection.
- **Playbook (PingZen health-check):** TCP-connect → TLS 1.3 ClientHello + HMAC-secret → валидация ServerHello + HMAC → Obfuscated2-Init в TLS-record → wait post-handshake response.
