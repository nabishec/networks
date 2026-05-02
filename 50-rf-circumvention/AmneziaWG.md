---
title: AmneziaWG
aliases: ["AWG", "AmneziaWireGuard"]
type: technique
layer: network
status: working
status_as_of: 2026-05-02
risk: medium
prerequisites: ["[[VPN]]", "[[Туннелирование]]"]
related: ["[[AmneziaVPN]]", "[[IPsec]]"]
sources: [src-02, src-06, src-07]
tags: [networking, rf-circumvention, technique]
---
# AmneziaWG (AWG)

## TL;DR
**Обфусцированный форк WireGuard** от команды Amnezia. Vanilla-WireGuard ловится DPI по **детерминированному handshake-pattern** (1-й пакет ровно 148 байт, magic-byte `0x01`); AmneziaWG добавляет **junk-packets** (случайной длины перед handshake) + меняет magic-байты + подменяет header-длины. Снаружи трафик выглядит как «случайный UDP», без вычислимого pattern.

## Какую проблему решает
- В РФ-2025 vanilla WireGuard детектируется и блокируется на DPI-уровне за **первый handshake-пакет**.
- IPsec/OpenVPN — давно ловятся.
- Reality/VLESS — TCP-based; UDP-альтернатива нужна для mobile (быстрее на lossy).
- AmneziaWG берёт скорость WireGuard + добавляет obfuscation минимально (без TLS-обёртки).

## Как работает

### Vanilla WireGuard handshake
1. Client → Server: 148 байт (magic=0x01, sender_index, ephemeral, encrypted_static, encrypted_timestamp, mac1, mac2).
2. Server → Client: 92 байта.

DPI читает первые 4 байта `0x01 0x00 0x00 0x00` → классифицирует как WireGuard.

### AmneziaWG модификации (S1, S2, H1, H2, H3, H4, Jc, Jmin, Jmax)
- **S1, S2** — добавляют random-bytes к handshake-пакетам.
- **H1, H2, H3, H4** — меняют magic-byte (вместо `0x01..0x04`).
- **Jc, Jmin, Jmax** — junk-packets перед handshake (Jc штук, длиной от Jmin до Jmax).

Конфиг `wg0.conf`:
```ini
[Interface]
PrivateKey = ...
Address = 10.10.0.2/24
Jc = 4
Jmin = 50
Jmax = 1000
S1 = 50
S2 = 100
H1 = 1234567890
H2 = 2345678901
H3 = 3456789012
H4 = 4567890123

[Peer]
PublicKey = ...
Endpoint = vpn.example.com:51820
```

DPI видит мусорный UDP-pattern → не классифицируется как WireGuard.

## Где ломается / почему может не работать
- **UDP-блокировка на whitelist-mobile** (src-05) — UDP/non-443 часто дропается.
- **Behavioral-detection:** даже obfuscated handshake выдаёт себя по timing/size-patterns на длительной сессии.
- **Несовместимость с upstream WireGuard** — нужны клиенты Amnezia.
- **Параметры (S1, S2, H1..H4) должны совпадать** у клиента и сервера — иначе handshake не пройдёт.

## Минимальный пошаговый сценарий
1. **Сервер:** установить через [[AmneziaVPN]]-deploy-script.
2. Скрипт автоматически генерирует уникальные `S1/S2/H1..H4/Jc/Jmin/Jmax`.
3. **Клиент:** AmneziaVPN (Win/Mac/Linux/iOS/Android) импортирует config.
4. Подключение — клиент шлёт junk-packets, потом modified-handshake.

## Что нужно
- VPS вне РФ (или внутри в whitelist-AS для cascade).
- AmneziaVPN-server (deploy через GUI клиента или CLI).
- Совместимый клиент (только Amnezia, не WireGuard-app).

## Связи
- **Базируется на:** [[VPN]] (концепт), [[Туннелирование]], **WireGuard** (ChaCha20-Poly1305 крипто).
- **Используется в:** [[AmneziaVPN]] (главный потребитель), self-hosted VPN-сервисы.
- **Соседи по уровню:** vanilla WireGuard (broken в РФ), [[VLESS-Reality]] (TCP-альтернатива), [[Shadowsocks-2022]].
- **Противопоставляется:** vanilla WireGuard — детектируется ТСПУ.

## Источники
- src-02, src-06, src-07.
