---
title: Shadowsocks-2022
aliases: ["SS2022"]
type: technique
layer: transport
status: partial
status_as_of: 2026-05-02
risk: medium
prerequisites: ["[[Симметричная vs асимметричная криптография]]", "[[TCP]]"]
related: ["[[VLESS-Reality]]", "[[uTLS]]"]
sources: [src-08]
tags: [networking, rf-circumvention, technique]
---
# Shadowsocks-2022

## TL;DR
Обновлённый стандарт **Shadowsocks** (2022, AEAD-2022 spec): фиксирует слабости старого SS (предсказуемые длины, replay attacks, weak handshake). Использует **AEAD-шифрование** (AES-GCM, ChaCha20-Poly1305) с **псевдослучайной обёрткой** в стиле случайных байтов. DPI видит «**случайный поток**» — нет SNI, нет сертификата, нет handshake. Хорош для не-РФ DPI; в РФ-2026 проигрывает Reality из-за «**активного зондирования**» (active probing): random-pattern узнаваем.

## Какую проблему решает
- Старый Shadowsocks был ловим из-за **постоянной длины handshake-frame** и predictable-pattern'ов.
- Новый SS2022:
  - **Random-padding** в кадрах.
  - **Replay-protection** через nonce-counter.
  - Лучшее поведение в **OBFS-плагинах** (cloak, simple-obfs).

## Как работает

**Структура соединения:**
1. Клиент: random-bytes-handshake (длина варьируется).
2. AEAD-шифрование с pre-shared key.
3. Внутри: SOCKS5-style tunnel.

**Ключевые ciphers:**
- `2022-blake3-aes-256-gcm` (рекомендация).
- `2022-blake3-chacha20-poly1305`.

## Где ломается / почему может не работать
- **Active probing:** DPI шлёт случайные пакеты на подозрительный сервер. Если сервер отвечает «как Shadowsocks» — заблокирован. Reality защищён target-сертификатом, SS — нет.
- **РФ-2025+:** SS-серверы регулярно обнаруживаются и попадают в blacklist.
- **Whitelist-модель** (src-09): random-IP/AS = блокировка по умолчанию.
- **Без TLS-маскировки** SS легко отличить от HTTPS по shape пакетов.

## Минимальный пошаговый сценарий
**Сервер (Shadowsocks-rust):**
```bash
ssserver -s "0.0.0.0:8388" -k "BASE64-PSK" -m "2022-blake3-aes-256-gcm"
```
**Клиент** (Hiddify, Nekobox): импорт ss://-link.

## Что нужно
- VPS (вне whitelist-РФ, но не критично где).
- Shadowsocks-rust ≥ v1.16.
- Клиенты с поддержкой 2022-spec.

## Связи
- **Базируется на:** [[Симметричная vs асимметричная криптография]] (AEAD), [[TCP]] (transport).
- **Соседи по уровню:** [[VLESS-Reality]] (более устойчив), Trojan (TLS-based), Hysteria (QUIC-based).
- **Противопоставляется:** Reality — Reality маскируется под TLS-сайт; SS просто «случайные байты».

## Источники
- src-08.
