---
title: Hysteria-2
aliases: ["Hysteria", "Hysteria2", "hysteria2"]
type: technique
layer: transport
status: partial
status_as_of: 2026-05-02
risk: medium
prerequisites: ["[[QUIC]]", "[[UDP]]", "[[TLS — рукопожатие]]"]
related: ["[[QUIC и mKCP]]", "[[mKCP]]", "[[VLESS-Reality]]", "[[AmneziaWG]]"]
sources: [src-08, src-06]
tags: [networking, rf-circumvention, technique]
---
# Hysteria-2 — QUIC-based VPN с custom congestion control

## TL;DR
**Hysteria-2** — VPN-протокол поверх **QUIC/UDP**, с собственным congestion-control-алгоритмом **Brutal**, маскировкой под HTTP/3 и опциональной обфускацией **Salamander** (XOR-обёртка Initial-пакетов). На lossy-каналах (5%+ packet-loss) даёт ~95% полосы там, где CUBIC-TCP проседает до ~30%. В РФ-2026 — **partial**: проходит, если AS/IP попадают в whitelist (Yandex/VK Cloud) и оператор не throttle'ит UDP. На мобильных вне whitelist — заблокирован на L3.

## Какую проблему решает
- **Session freezing** ([[Session freezing]]): TCP-VPN в РФ DPI замораживает после ~16 КБ. QUIC-поток состоит из независимых datagram'ов → нет долгого state, который DPI мог бы заморозить.
- **Mobile lossy radio:** TCP CUBIC плохо адаптируется к 5–10% packet-loss; Brutal-congestion в Hysteria держит throughput близко к bandwidth-cap.
- **Маскировка под HTTP/3:** YouTube/Google-сервисы массово ходят по QUIC/443 → DPI не может срезать «всё подряд UDP/443».

## Как работает
1. **Транспорт:** QUIC-v1 (RFC 9000) поверх UDP/443.
2. **Handshake:** TLS 1.3 + ALPN `h3` → внешне = HTTP/3. Сертификат — обычно Let's Encrypt на свой домен.
3. **Congestion:** **Brutal** — клиент сам декларирует upload/download bandwidth-cap; сервер шлёт ровно столько, без classical AIMD-ramp-up.
4. **Аутентификация:** password (HMAC) или OBFS-секрет.
5. **Salamander** (опц.): XOR-обёртка Initial-пакетов с pre-shared password → Initial выглядит как random bytes → DPI не находит QUIC-magic.

## Где ломается / почему может не работать
- **UDP-блокировка / throttling:** некоторые мобильные операторы режут UDP до 100–500 Кбит/с (особенно non-443).
- **Whitelist на UDP/443:** DPI пропускает QUIC только к Google/Cloudflare/Yandex. Свой VPS вне whitelist → дроп.
- **Active probing:** атакующий шлёт фальшивый QUIC-Initial → если сервер ответит на любой UDP — раскрытие. Salamander-обфускация частично помогает (нет ответа без правильного password).
- **NAT-timeout:** мобильный NAT держит UDP-flow ~30 с. Без keep-alive сессия рвётся → нужен `quic.keepAlivePeriod`.
- **Нет панелей:** Hysteria-2 не имеет полноценной web-панели, в отличие от [[3X-UI]] / [[Marzban]] для Xray. Управление — через config-файл.

## Минимальный пошаговый сценарий

**Сервер (Debian/Ubuntu):**
```bash
bash <(curl -fsSL https://get.hy2.sh/)
```

```yaml
# /etc/hysteria/config.yaml
listen: :443
acme:
  domains: [my.example.com]
  email: me@example.com
auth:
  type: password
  password: "STRONG-PASSWORD"
masquerade:
  type: proxy
  proxy:
    url: https://news.ycombinator.com  # реальный сайт-маскировщик для не-аутентифицированных
    rewriteHost: true
obfs:
  type: salamander
  salamander:
    password: "OBFS-SECRET"            # рекомендуется в РФ-2026
```

**Запуск:** `systemctl enable --now hysteria-server`.

**Клиент:** Hiddify-Next / NekoBox / v2rayN — импорт `hysteria2://STRONG-PASSWORD@my.example.com:443/?obfs=salamander&obfs-password=OBFS-SECRET#myhy`.

## Что нужно
- VPS с public IP, открытым **UDP/443**.
- Свой домен + LE-сертификат (для маскировки и ACME-автообновления).
- Hysteria-2 ≥ v2.5 (Salamander).
- Клиент с поддержкой Hysteria-2 (Hiddify, NekoBox, FoxRay, v2rayN).
- В whitelist-режиме — VPS в **разрешённом AS** (Yandex Cloud AS 13238, VK Cloud).

## Связи
- **Базируется на:** [[QUIC]] (транспорт), [[UDP]], [[TLS — рукопожатие]].
- **Сравнивается с:** [[mKCP]] (тот же UDP, но Xray-transport, без отдельного congestion-алгоритма), [[VLESS-Reality]] (TCP-альтернатива, лучше в whitelist-сетях, где UDP блокирован).
- **Альтернативы:** Tuic (другая QUIC-VPN), AmneziaWG (UDP, но WireGuard-base).
- **Сводка:** [[QUIC и mKCP]].
- **Клиенты:** [[Hiddify]] поддерживает «из коробки».

## Источники
- src-08 — обзор протоколов 2024, явно упоминает QUIC/mKCP/Hysteria.
- src-06 — Encrypted-каскад с Hysteria-нодой.
- [Hysteria 2: протокол, который притворяется HTTP/3 и почти не врёт — habr 1008554](https://habr.com/ru/articles/1008554/)
- [Установка и настройка Hysteria — habr 990176](https://habr.com/ru/articles/990176/)
- [Установка и настройка VPN-сервера на протоколе Hysteria 2 — habr sandbox 279400](https://habr.com/ru/sandbox/279400/)
