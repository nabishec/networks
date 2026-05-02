---
title: "PB7 — basic VLESS-Reality с нуля"
aliases: ["PB7"]
type: playbook
layer: application
status: working
status_as_of: 2026-05-02
risk: medium
prerequisites: ["[[VLESS-Reality]]", "[[Xray-core]]", "[[TLS — рукопожатие]]"]
related: ["[[Hiddify]]", "[[Диффи-Хеллман]]"]
sources: [src-08]
tags: [networking, rf-circumvention, playbook]
---
# PB7 — basic VLESS-Reality с нуля

## TL;DR
Канонический «базовый» рецепт по src-08 (март 2024): VPS вне РФ + Xray + VLESS-Reality + клиент Hiddify/Nekoray. **Самое простое** что работает в РФ-2026 на не-whitelist-провайдерах (домашний интернет, обычный home Wi-Fi).

## Шаги

### 1. Купить VPS вне РФ
Bezel-минимум:
- 1 vCPU, 1 GB RAM, 25 GB disk.
- Linux (Ubuntu 22.04 / Debian 12).
- Public IPv4.
- Trafic 1-5 TB/мес.

Провайдеры: Hetzner (CX11 ~3€/мес), OVH, DigitalOcean ($5), Vultr.

### 2. Выбрать target-сайт для маскировки
- TLS 1.3 + h2.
- Не банальный `google.com` (заблокирован в Reality в некоторых клиентах) — лучше `microsoft.com`, `apple.com`, `github.com`, `cloudflare.com`.

Проверка:
```bash
curl -I --tlsv1.3 https://microsoft.com 2>&1 | head -5
# Должно быть TLSv1.3 + ALPN h2.
```

### 3. Установить Xray
```bash
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install
```

### 4. Сгенерировать ключи
```bash
xray x25519
# Output:
# Private key: SERVER_PRIV
# Public key: SERVER_PUB

xray uuid
# Output: UUID-CLIENT

# Short-id (random hex 8 chars):
openssl rand -hex 8
# Output: SHORTID
```

### 5. Конфиг `/usr/local/etc/xray/config.json`
```json
{
  "log": { "loglevel": "warning" },
  "inbounds": [{
    "port": 443,
    "protocol": "vless",
    "settings": {
      "clients": [{ "id": "UUID-CLIENT", "flow": "xtls-rprx-vision" }],
      "decryption": "none"
    },
    "streamSettings": {
      "network": "tcp",
      "security": "reality",
      "realitySettings": {
        "show": false,
        "dest": "microsoft.com:443",
        "xver": 0,
        "serverNames": ["microsoft.com", "www.microsoft.com"],
        "privateKey": "SERVER_PRIV",
        "shortIds": ["SHORTID"]
      }
    },
    "sniffing": { "enabled": true, "destOverride": ["http", "tls"] }
  }],
  "outbounds": [
    { "protocol": "freedom", "tag": "direct" },
    { "protocol": "blackhole", "tag": "blocked" }
  ]
}
```

### 6. Перезапустить
```bash
systemctl enable --now xray
systemctl status xray  # должен быть active
```

### 7. Собрать VLESS-link
```
vless://UUID-CLIENT@SERVER-IP:443?security=reality&encryption=none&pbk=SERVER_PUB&sid=SHORTID&type=tcp&flow=xtls-rprx-vision&sni=microsoft.com&fp=chrome#MyServer
```

### 8. Установить клиент
- **Windows/macOS/Linux:** Hiddify-Next, Nekoray.
- **Android:** v2rayNG, Hiddify, NekoboxForAndroid.
- **iOS:** Streisand, Shadowrocket, FoxRay, v2raytun, Happ.

Импорт link'а или QR.

## Проверка
- В клиенте: подключиться, открыть `https://ifconfig.me` → IP сервера, не локальный.
- `https://browserleaks.com/ip` — посмотреть, нет ли утечек.
- `https://www.dnsleaktest.com/` — DNS-leak.

## Где ломается
- **Whitelist-провайдер** (мобильный РФ) — IP сервера не в whitelist → не работает. Нужен [[PB1 — Yandex API Gateway фронтинг]] или [[PB2 — vnext-цепочка через РФ-мост]].
- **target-сайт заблокирован** в РФ → handshake не пройдёт. Сменить target.
- **DPI с поведенческим анализом** — может детектировать через packet-shape.

## Связи
- **Технический фундамент:** [[VLESS-Reality]], [[XTLS-Vision]], [[Xray-core]], [[Hiddify]].
- **Расширения:** [[PB2 — vnext-цепочка через РФ-мост]] (для whitelist-провайдеров), [[PB5 — РФ-каскад с xHTTP+packet-up]] (для session-freezing).

## Источники
- src-08.
