---
title: Xray-core
aliases: ["Xray", "xray-core"]
type: tool
layer: application
status: working
status_as_of: 2026-05-02
risk: low
prerequisites: ["[[Туннелирование]]", "[[VPN]]"]
related: ["[[VLESS-Reality]]", "[[xHTTP]]", "[[XTLS-Vision]]", "[[Sing-box]]", "[[TLS — рукопожатие]]"]
sources: [src-01, src-02, src-03, src-06, src-08]
tags: [networking, rf-circumvention, tool]
---
# Xray-core

## TL;DR
**Главный** open-source proxy-engine 2024-2026 для обхода блокировок. Форк V2Ray. Реализует семейство протоколов: **VLESS, VMess, Trojan, Shadowsocks-2022, Reality, XTLS-Vision, xHTTP, гибрид'ы**. Поддерживает routing с geo-файлами, multiplexing, кастомные транспорты. Используется как сервер и клиент. Минимальная актуальная версия для РФ-2026: **≥ v25.12.8** (src-02).

## Что делает
- **Server:** принимает inbound (VLESS/Trojan/SS) + проксирует через outbound (Reality, vnext, freedom).
- **Client:** обратное.
- **Routing:** правила по domain/IP/geo для split tunneling.
- **Mux:** multiplex много sessions через одно TCP-соединение.

## Установка (server, Linux)
```bash
# Officia install-script:
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install

# Минимальный config /usr/local/etc/xray/config.json (VLESS-Reality):
{
  "inbounds": [{
    "port": 443, "protocol": "vless",
    "settings": { "clients": [{ "id": "UUID", "flow": "xtls-rprx-vision" }] },
    "streamSettings": {
      "network": "tcp", "security": "reality",
      "realitySettings": {
        "dest": "microsoft.com:443",
        "serverNames": ["microsoft.com"],
        "privateKey": "PRIV", "shortIds": ["abc"]
      }
    }
  }],
  "outbounds": [{ "protocol": "freedom" }]
}
```

```bash
xray x25519  # generate keys
systemctl enable --now xray
```

## Где взять / документация
- GitHub: https://github.com/XTLS/Xray-core
- Документация: https://xtls.github.io/

## Альтернативы
- **[[Sing-box]]** — Go-проект, более модульный, поддерживает почти то же.
- **V2Ray** — родитель, менее активен.
- **Mihomo** (Clash-fork) — для клиентов.

## Связи
- **Используется в:** [[VLESS-Reality]], [[xHTTP]], [[XTLS-Vision]], [[Self-Steal — свой домен]], [[Split routing]], [[vnext-цепочка]].
- **Соседи по уровню:** [[Sing-box]] (альтернатива), Mihomo, V2Ray.
- **Управляется панелями:** [[3X-UI и Marzban]], Remnawave.

## Источники
- src-01, src-02 (≥v25.12.8 для xHTTP), src-03, src-06, src-08.
