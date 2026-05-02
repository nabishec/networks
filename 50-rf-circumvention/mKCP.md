---
title: mKCP
aliases: ["mKCP", "Xray mKCP transport"]
type: technique
layer: transport
status: partial
status_as_of: 2026-05-02
risk: medium
prerequisites: ["[[UDP]]", "[[Xray-core]]"]
related: ["[[QUIC и mKCP]]", "[[Hysteria-2]]", "[[VLESS-Reality]]", "[[xHTTP]]"]
sources: [src-08]
tags: [networking, rf-circumvention, technique]
---
# mKCP — Xray reliable-UDP transport

## TL;DR
**mKCP** — это **transport** Xray/V2Ray (не самостоятельный proxy-протокол), реализующий **reliable-UDP** на базе KCP с FEC и адаптивным retransmit. Поверх mKCP всегда едет **VLESS / VMess / Trojan**. Полезен там, где TCP плохо себя ведёт (high-latency, lossy radio), и где нужно **замаскировать UDP под BitTorrent/SRTP/uTP/WeChat/DTLS** (опция `header.type`). В РФ-2026 — **partial**: с правильным `seed` и header-маскировкой иногда проходит non-whitelist-сети, но в whitelist-mobile UDP к чужим IP всё равно дропается. С v25.x в Xray есть отчёты о деградации (CPU-spike, throughput-throttle) на некоторых сетях.

## Какую проблему решает
- **TCP session-freezing** ([[Session freezing]]): mKCP — UDP, нет долгого TCP-state.
- **Lossy mobile radio:** KCP-FEC выгоднее «голого» TCP при packet-loss > 3%.
- **Маскировка UDP-flow:** `header.type=wechat-video` / `srtp` / `utp` — DPI видит «WeChat-видеозвонок», а не VLESS-payload.
- **Низкий overhead** vs QUIC: mKCP проще и легче, без TLS/handshake (TLS-обёртку можно добавить отдельно).

## Как работает
1. **KCP** — алгоритм reliable-UDP «на уровне приложения»: ARQ + FEC + congestion control (но проще, чем у QUIC).
2. **modified KCP (mKCP)** в V2Ray/Xray: добавлена **header-обфускация** (имитация известных UDP-протоколов), **seed** (pre-shared-ключ для контроль-пакетов), `congestion=true` для адаптации к потерям.
3. **Поверх** — обычный VLESS/VMess; mKCP отвечает только за reliable-UDP-транспорт между клиентом и сервером.

## Где ломается / почему может не работать
- **Whitelist-mobile (src-05):** UDP к чужим IP блокируется на L3 → mKCP вообще не доходит до сервера.
- **Active probing:** без `seed` любой может послать KCP-пакет, и сервер ответит → детект. С `seed` устойчивее.
- **Header-обёртки `srtp`/`wechat-video` устаревают:** DPI знает реальные SRTP-flow и видит, что mKCP-«srtp» отличается → детект по deeper-fingerprint.
- **Деградация в новых Xray-релизах:** на некоторых сетях с v25.x mKCP начал throttling/CPU-spike (см. сообщения в комментариях Habr 807163). Без A/B-теста не использовать продакшн.
- **Уступает Hysteria-2** по UX и congestion-control: у mKCP нет Brutal-алгоритма; на 5% loss проседает заметнее.

## Минимальный пошаговый сценарий

**Сервер (Xray inbound):**
```json
{
  "inbounds": [{
    "port": 8443,
    "protocol": "vless",
    "settings": { "clients": [{ "id": "UUID-HERE" }], "decryption": "none" },
    "streamSettings": {
      "network": "mkcp",
      "kcpSettings": {
        "mtu": 1350,
        "tti": 50,
        "uplinkCapacity": 50,
        "downlinkCapacity": 100,
        "congestion": true,
        "header": { "type": "wechat-video" },
        "seed": "MY-SHARED-SEED"
      }
    }
  }]
}
```

**Клиент (Hiddify / Nekobox):** импорт VLESS-link с `type=kcp&headerType=wechat-video&seed=MY-SHARED-SEED`.

**Маскировка под BitTorrent / FaceTime:**
- `header.type=utp` — uTorrent.
- `header.type=srtp` — FaceTime/WebRTC.
- `header.type=dtls` — DTLS-handshake-mock.

## Что нужно
- Xray-core ≥ v1.8 (рекомендуется свежий релиз без mKCP-регрессий).
- VPS с открытым UDP-портом (любой, не обязательно 443).
- Совпадающие `seed` и `header.type` на клиенте/сервере.
- Клиент с mKCP (Xray-clients: Hiddify, NekoBox, v2rayN, v2rayNG).

## Связи
- **Базируется на:** [[UDP]], [[Xray-core]] (только в Xray/V2Ray-стеке).
- **Сравнивается с:** [[Hysteria-2]] — Hysteria даёт лучше congestion и маскировку под HTTP/3; mKCP проще и не требует TLS/домена.
- **Соседи в Xray:** [[xHTTP]] (TCP-альтернатива против session-freezing), WebSocket, gRPC.
- **Сводка:** [[QUIC и mKCP]].

## Источники
- src-08 — упоминание QUIC/mKCP среди транспортов.
- [Настройка протокола mKCP в 3X-UI и X-UI для маскировки трафика — habr 807163](https://habr.com/ru/articles/807163/)
- [Современные технологии обхода блокировок: V2Ray, XRay, XTLS, Hysteria, Cloak — habr 727868](https://habr.com/ru/articles/727868/)
- [Малоизвестные фичи XRay — habr 834290](https://habr.com/ru/articles/834290/)
