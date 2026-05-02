---
title: "PB4 — диагностика whitelist"
aliases: ["PB4"]
type: playbook
layer: cross-layer
status: working
status_as_of: 2026-05-02
risk: low
prerequisites: ["[[Белые списки]]", "[[ТСПУ]]", "[[DNS]]", "[[TLS — рукопожатие]]"]
related: ["[[DPI-фильтрация в РФ]]", "[[ICMP]]", "[[NAT]]"]
sources: [src-05]
tags: [networking, rf-circumvention, playbook]
---
# PB4 — диагностика whitelist на провайдере

## TL;DR
Серия диагностических команд (по src-05) для понимания, **какой режим фильтрации** на твоём провайдере: blacklist / whitelist / mixed. Без диагностики можно применить ненужную технику и думать, что не работает.

## Шаги

### 1. DNS — резолв через разные resolver'ы
```bash
# Yandex DNS (обычно разрешён в РФ-whitelist):
dig @77.88.8.8 microsoft.com

# Cloudflare DNS:
dig @1.1.1.1 microsoft.com

# Google DNS (часто блокирован на мобильных в РФ):
dig @8.8.8.8 microsoft.com
```
- Если Yandex отвечает, остальные timeout/refused → **whitelist DNS**.
- Если все отвечают, но Cloudflare с другим IP → **DNS spoofing**.

### 2. ICMP — ping
```bash
ping -c 3 microsoft.com  # ping IPv4
ping -c 3 1.1.1.1
```
ICMP rate-limiting на провайдере — норма, но 100% потери = блокировка ICMP.

### 3. HTTPS — curl с timeout
```bash
curl https://microsoft.com -m 5 -o /dev/null -w "%{http_code} %{time_connect} %{size_download}\n"
curl https://github.com -m 5 -o /dev/null -w "%{http_code} %{time_connect} %{size_download}\n"
curl https://twitter.com -m 5 -o /dev/null -w "%{http_code} %{time_connect} %{size_download}\n"
```
- 200 + быстро → разрешено.
- timeout без RST → blacklist (silent drop) или whitelist-block.
- size_download = 16384 → [[Session freezing]].

### 4. Traceroute — маркер ТСПУ
```bash
traceroute -n microsoft.com
```
Ищи `198.18.x.x` или `100.64.x.x` (CGNAT) — маркер ТСПУ-карты или CGNAT мобильного оператора.

### 5. Wireshark — анализ TLS-handshake
```bash
sudo tcpdump -i any -w capture.pcap host microsoft.com
# (открыть в Wireshark, фильтр: tls.handshake)
```
- SYN ушёл, SYN-ACK не пришёл → IP-blacklist на L3.
- ClientHello ушёл, ответа нет → SNI-фильтрация.
- ClientHello + ServerHello, но потом 16 KB и stop → session-freezing.

### 6. mtr — маршруты с потерями
```bash
mtr -c 50 microsoft.com
```
Где первые потери — там фильтрующий узел.

## Интерпретация

| Симптом | Вероятная причина |
|---|---|
| DNS у Yandex работает, у других нет | Whitelist DNS |
| Ping 100% loss, HTTPS работает | ICMP-rate-limit (норма) |
| HTTPS timeout без RST | IP-blacklist или silent drop |
| 16 KB и stop | Session freezing |
| ClientHello → silent | SNI-blacklist |
| 198.18.x.x в traceroute | ТСПУ |
| 100.64.x.x | CGNAT |
| `dig @8.8.8.8` пустой | Whitelist-DNS на оператор-уровне |

## Что нужно
- Linux/macOS-shell или WSL.
- `dig`, `curl`, `traceroute`, `mtr`, `tcpdump`/Wireshark.

## Связи
- **Технический фундамент:** [[Белые списки]], [[SNI-фильтрация]], [[Session freezing]], [[ТСПУ]], [[DPI-фильтрация в РФ]].
- **Использует инструменты:** Wireshark, dig, ping, curl, traceroute, tcpdump, mtr.

## Источники
- src-05.
