---
title: Sing-box
aliases: ["sing-box"]
type: tool
layer: application
status: working
status_as_of: 2026-05-02
risk: low
prerequisites: ["[[Туннелирование]]"]
related: ["[[Xray-core]]", "[[Hiddify]]", "[[TLS — рукопожатие]]"]
sources: [src-01, src-08]
tags: [networking, rf-circumvention, tool]
---
# Sing-box

## TL;DR
Modular **Go-based proxy** от автора SagerNet. Альтернатива Xray с более чистым кодом и интегрированным DNS-resolver, **rule-engine** для split routing, поддержкой множества протоколов (VLESS, Reality, Trojan, Shadowsocks, Hysteria-2, Tuic, Naive). Главный движок **Hiddify-Next** и других modern-клиентов.

## Что делает
- Сервер + клиент в одном бинаре.
- **DNS-resolver** встроенный (с DoH/DoT/forwarding).
- **Rule-engine** — мощнее Xray-routing.
- **TUN-mode** — клиент работает на L3 (как VPN), не SOCKS5.
- **Inbound types:** mixed (HTTP+SOCKS5), TUN, redirect.

## Конфиг (фрагмент)
```json
{
  "outbounds": [{
    "type": "vless",
    "tag": "proxy",
    "server": "example.com",
    "server_port": 443,
    "uuid": "UUID",
    "tls": { "enabled": true, "reality": { ... } }
  }],
  "route": {
    "rules": [
      { "geosite": ["category-ru"], "outbound": "direct" },
      { "outbound": "proxy" }
    ]
  }
}
```

## Где взять
- GitHub: https://github.com/SagerNet/sing-box
- Документация: https://sing-box.sagernet.org/

## Связи
- **Используется в:** [[Hiddify]] (Hiddify-Next built on sing-box), Karing, NekoBox.
- **Соседи по уровню:** [[Xray-core]] (главный конкурент).
- **Поддерживает:** [[VLESS-Reality]], [[xHTTP]], [[Shadowsocks-2022]], [[QUIC и mKCP]] (Hysteria, Tuic).

## Источники
- src-01, src-08.
