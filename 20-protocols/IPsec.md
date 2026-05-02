---
title: IPsec
aliases: ["IP Security", "AH", "ESP", "IKE"]
type: protocol
layer: security
chapter: 8
difficulty: intermediate
prerequisites: ["[[IPv4]]", "[[Симметричная vs асимметричная криптография]]"]
related: ["[[VPN]]", "[[Туннелирование]]"]
tags: [networking, ch08]
---
# IPsec — IP Security (RFC 4301+)

## TL;DR
Семейство протоколов для **шифрования и аутентификации IP-пакетов**. Главные части: **AH** (Authentication Header — только integrity/auth), **ESP** (Encapsulating Security Payload — encryption + auth), **IKE** (Internet Key Exchange — установление ключей). Два режима: **transport** (только payload шифруется) и **tunnel** (весь IP-пакет шифруется и упаковывается в новый — для VPN). Базис большинства IPsec-VPN.

## Какую проблему решает
До TLS на L7, нужно было защищать **весь IP-трафик** между сетями (corporate VPN). IPsec работает на L3 — прозрачно для приложений. Любое приложение, шлющее IP, защищается без изменений.

## Как работает

### AH (Authentication Header, RFC 4302)
- Только **integrity + authentication**.
- HMAC-based MAC всего IP-пакета.
- Не шифрует payload.
- Редко используется отдельно (без шифрования).

### ESP (Encapsulating Security Payload, RFC 4303) — главный
- Шифрует payload + опционально authentic'ит.
- Modern: AES-GCM (AEAD) — both в одном.
- Старые AES-CBC + HMAC-SHA-256.

### Transport mode vs Tunnel mode

| | Transport | Tunnel |
|---|---|---|
| Что шифруется | только payload IP-пакета | весь IP-пакет (включая заголовок) |
| Новый IP-заголовок | нет | да (от GW к GW) |
| Применение | host-to-host | site-to-site VPN, remote access |

**Tunnel mode пример:**
```
Original: [IP1 | TCP | data]
After IPsec tunnel:
[New IP (GW1→GW2) | ESP header | encrypted(IP1 | TCP | data) | ESP trailer | ICV]
```

### IKE (Internet Key Exchange, RFC 7296 — IKEv2)
- Phase 1: устанавливает **IKE SA** (security association) — secure channel между peers (Diffie-Hellman + auth).
- Phase 2: устанавливает **IPsec SAs** для actual data.
- Auth: **PSK** (pre-shared key), **certificates**, **EAP**.
- IKEv2 — современный, проще IKEv1.

**SA** — Security Association: tuple (algorithm, key, SPI). Identifies one direction of secure communication.

## Пример
**Site-to-site VPN между офисами:**
- Gateway1 (Москва, public IP 1.1.1.1) ↔ Gateway2 (Питер, 2.2.2.2).
- IKEv2 устанавливает IPsec SAs.
- Tunnel mode ESP с AES-256-GCM.
- Весь трафик 10.1.0.0/16 ↔ 10.2.0.0/16 шифруется на стыке.
- Снаружи виден только обмен IP-пакетов между 1.1.1.1 и 2.2.2.2.

**Remote access:**
- Сотрудник дома → IPsec-клиент (или WireGuard/OpenVPN) → corporate GW.
- Tunnel mode, RA подобный VPN.

## Связи
- **Базируется на:** [[IPv4]], [[IPv6]] (extension headers), [[Симметричная vs асимметричная криптография]], [[Диффи-Хеллман]] (в IKE).
- **Используется в:** site-to-site VPN, mobile carrier core (S8 between MMEs), межоператорские VPN.
- **Соседи по уровню:** **WireGuard** — современный простой VPN (UDP, ChaCha20), **OpenVPN** — TLS-based, **TLS** — на L7.
- **Противопоставляется:** TLS — на L7, специфично для протокола; IPsec — на L3, протокол-нейтрален.

## Подводные камни
- **NAT traversal:** IPsec-ESP не имеет ports → NAT не может переписать. Решение — **NAT-T** (UDP encapsulation port 4500).
- **Сложная конфигурация:** IKE имеет десятки параметров (DH groups, lifetimes, encryption suites) — ошибки приводят к нерабочему туннелю.
- **WireGuard простота** становится новым стандартом для VPN — IPsec остаётся в legacy и enterprise.
- **MTU issues:** IPsec добавляет header → effective MTU < 1500. PMTUD ломается → fragmentation.

## Дальше читать
- [[VPN]] — общий концепт.
- [[Туннелирование]] — родственная тема.
- Tanenbaum, гл. 8, §8.10.1 (стр. PDF 911–915).
