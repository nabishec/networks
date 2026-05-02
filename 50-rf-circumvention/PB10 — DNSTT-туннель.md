---
title: "PB10 — DNSTT-туннель"
aliases: ["PB10", "DNSTT playbook"]
type: playbook
layer: application
status: partial
status_as_of: 2026-05-02
risk: low
prerequisites: ["[[DNS-туннелирование]]", "[[DNS]]", "[[DNS — типы записей]]"]
related: ["[[Encrypted DNS — DoH-DoT]]", "[[PingTunnel — ICMP]]", "[[Белые списки]]"]
sources: [src-01, src-08]
tags: [networking, rf-circumvention, playbook]
---
# PB10 — DNSTT-туннель (last-resort через DNS)

## TL;DR
Когда **WebRTC, MTProxy и Reality** уже не работают (например, mobile-whitelist + блокировка Yandex API), остаётся **DNS** — операторы редко его режут целиком, иначе ломается резолвинг. **DNSTT** (David Fifield, 2020) гонит TCP-стрим через DoH-резолвер (Cloudflare/Google) → authoritative-nameserver под нашим контролем → backend. Реальная скорость **1.5-3 Mbps** (Habr-757420), latency десятки секунд — не для bulk, но для текстовых каналов и Telegram-API хватает.

## Когда брать
- DNS — **последняя** не отключенная transport-плоскость.
- Mobile-whitelist режет всё, кроме DoH к whitelisted-резолверу.
- Нужен только **доступ** (e-mail, мессенджер), не throughput.

## Архитектура
```mermaid
flowchart LR
  C[Client dnstt-client] -->|"DoH https://1.1.1.1/dns-query"| Resolver[Cloudflare/Yandex]
  Resolver -->|"NS-delegation tunnel.example.com"| Auth[VPS: dnstt-server :5300/udp]
  Auth -->|"localhost:8080"| Backend[SOCKS5/SSH/HTTP-proxy]
```

## Шаги

### 1. Зарегистрировать домен
Нужен **обычный домен** (`example.com`) и поддомен под туннель (`tunnel.example.com`).

### 2. Настроить DNS-делегирование
В DNS-провайдере домена создать **NS-delegation** для поддомена:
```
tunnel.example.com.    NS    ns1.example.com.
ns1.example.com.       A     SERVER-IP
```
Это говорит резолверу: «за `*.tunnel.example.com` спрашивай SERVER-IP». **Glue-record** обязательно — без него резолвер не найдёт `ns1`.

### 3. Открыть UDP/53 на сервере
```bash
ufw allow 53/udp
# Если резолвер systemd-resolved уже занял :53 — выключить или перенаправить:
systemctl disable --now systemd-resolved
# DNSTT удобнее запустить на :5300 + iptables redirect:
iptables -t nat -A PREROUTING -p udp --dport 53 -j REDIRECT --to-port 5300
```

### 4. Установить DNSTT
```bash
git clone https://www.bamsoftware.com/git/dnstt.git
cd dnstt/dnstt-server && go build
cd ../dnstt-client && go build
```

### 5. Сгенерировать ключи и запустить server
```bash
./dnstt-server -gen-key -privkey-file server.key -pubkey-file server.pub
# Запустить:
./dnstt-server -udp :5300 -privkey-file server.key tunnel.example.com 127.0.0.1:8080
```
Backend на `127.0.0.1:8080` — обычно `dante` SOCKS5 или `sshd` (через SSH-tunnel SOCKS5 заворачивается клиентом).

### 6. systemd-unit
```ini
[Unit]
Description=dnstt-server
After=network.target

[Service]
ExecStart=/usr/local/bin/dnstt-server -udp :5300 -privkey-file /etc/dnstt/server.key tunnel.example.com 127.0.0.1:8080
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

### 7. Запустить клиент
На клиенте:
```bash
dnstt-client -doh https://1.1.1.1/dns-query \
  -pubkey-file server.pub \
  tunnel.example.com 127.0.0.1:7000
```
Локальный `127.0.0.1:7000` — TCP-стрим к серверу через DNS. Поверх — `ssh -D 9050 -p 7000 user@127.0.0.1` или прямой SOCKS5.

### 8. Через Yandex DNS (whitelist-режим)
В жёстком РФ-mobile может работать только Yandex DoH:
```bash
dnstt-client -doh https://common.dns.yandex.net/dns-query ...
```
Тогда Yandex выступает рекурсором → пойдёт к нашему authoritative; работает, если authoritative-IP не в blacklist.

## Проверка
- `dig @SERVER-IP tunnel.example.com TXT` — должен вернуть base32-payload.
- `curl --socks5 127.0.0.1:9050 https://ifconfig.me` через клиента → IP сервера.
- Скорость: `time curl -o /dev/null https://example.com/1mb.bin` — ожидать 30-60 сек на 1 МБ.

## Где ломается
- **Скорость** 1-3 Mbps **в идеале**; реально 100-300 Kbps. Веб-браузинг — мучение, но Telegram-API/SSH-shell идёт.
- **Latency** каждого «пакета» — сотни мс (DoH-RTT × encoding overhead).
- **DPI на DoH-pattern.** Если резолвер видит **много запросов с длинными labels** к одному домену — может маркировать. Решение: разнести на несколько subdomain'ов, ввести jitter.
- **Whitelist-DNS** (src-05): mobile-операторы могут разрешить только Yandex 77.88.8.8 → DoH к Cloudflare/Google не пройдёт. Использовать `https://common.dns.yandex.net/dns-query`.
- **Authoritative-IP попал в blacklist** — резолверы перестают форвардить. Сменить IP / AS.
- **iOS** — клиента нет; на Android только через TLS Tunnel-плагин.

## Связи
- **Технический фундамент:** [[DNS-туннелирование]], [[DNS]], [[DNS — типы записей]] (TXT/AAAA), [[Encrypted DNS — DoH-DoT]].
- **Соседи по уровню:** [[PingTunnel — ICMP]] (другой last-resort), [[WebRTC-туннель]] (если UDP-видеозвонки разрешены).
- **Альтернатива при наличии WebRTC:** [[PB1 — Yandex API Gateway фронтинг]] — быстрее на порядки.
- **Используется в:** Snowflake (Tor-bridge с DNSTT-fallback).

## Источники
- src-01 (xDNS как один из 6 способов обхода whitelist), src-08 (DNSTT как last-resort 2024).
- Habr: [DNSTT. DNS туннель для обхода блокировок](https://habr.com/ru/articles/757420/) — архитектура, NS-delegation, DoH-резолверы, скорость 1.5 Mbps.
