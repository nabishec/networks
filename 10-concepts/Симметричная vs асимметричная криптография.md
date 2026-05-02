---
title: Симметричная vs асимметричная криптография
aliases: ["Symmetric", "Asymmetric", "Public key crypto"]
type: concept
layer: security
chapter: 8
difficulty: basic
prerequisites: []
related: ["[[DES и AES]]", "[[RSA]]", "[[Диффи-Хеллман]]"]
tags: [networking, ch08]
---
# Симметричная vs асимметричная криптография

## TL;DR
**Симметричная**: один секретный ключ для шифрования и расшифровки (AES, ChaCha20). Быстрая, но ключ нужно как-то **передать обеим сторонам**. **Асимметричная**: пара ключей — **публичный** (распространяется) и **приватный** (хранится). Что зашифровано публичным — расшифрует только приватный, и наоборот для подписей. Медленная (RSA, ECC), но решает проблему key exchange. **На практике гибрид:** асимметричная для обмена ключом, симметричная для собственно шифрования.

## Какую проблему решает
До 1976 г. была только симметричная криптография — две стороны как-то должны обменяться секретом. **Diffie & Hellman (1976)** показали: можно через open channel установить общий секрет. **RSA (1978)** добавил отдельную пару ключей, public-key. Это перевернуло криптографию — теперь любой может зашифровать сообщение для вас, не зная вашего секрета.

## Как работает

| | Симметричная | Асимметричная |
|---|---|---|
| Ключи | один общий секрет | пара (public + private) |
| Скорость | гигабайты/с (AES-NI) | килобайты/с (RSA-2048) |
| Размер ключа | 128–256 бит | RSA 2048+ бит, ECC 256 бит |
| Алгоритмы | AES, ChaCha20, DES (устар.) | RSA, ECDSA, Ed25519, ECDH |
| Применение | bulk encryption | key exchange, подписи |
| Key distribution | сложно (как передать?) | легко (public — публикуй) |

**Гибридная схема (TLS, SSH):**
1. Asymmetric handshake: установить общий symmetric session key.
2. Дальнейшие данные — symmetric с этим ключом.
3. Скорость symmetric + удобство key distribution.

## Пример
**TLS-handshake** ([[TLS — рукопожатие]]):
- ClientHello/ServerHello.
- ECDHE: ephemeral Diffie-Hellman → общий секрет.
- Из секрета derive symmetric keys (AES-256-GCM).
- Все application data — AES.

**RSA-шифрование email** (PGP):
- Сообщение → шифровать AES random session key.
- Session key → шифровать получательским RSA public key.
- Получатель: RSA private decrypt session key → AES decrypt тело.

## Связи
- **Базируется на:** mathematics (number theory, ECC).
- **Используется в:** [[DES и AES]], [[RSA]], [[Диффи-Хеллман]] — конкретные алгоритмы; вся современная security.
- **Соседи по уровню:** **post-quantum cryptography** — устойчивая к квантовым компьютерам (Kyber, Dilithium).
- **Противопоставляется:** plaintext — никакой защиты.

## Подводные камни
- **Symmetric without key exchange** не помогает в реальном мире — асимметрия неизбежна.
- **Asymmetric для большого payload** — слишком медленно. Hybrid обязателен.
- **RSA-2048 vs ECC-256** — equivalent security, но ECC дешевле.
- **Quantum threat:** RSA и DH ломаются Шора-алгоритмом → **post-quantum** crypto в стандартизации (NIST).

## Дальше читать
- [[DES и AES]], [[RSA]], [[Диффи-Хеллман]] — конкретные.
- [[TLS — рукопожатие]] — гибрид в действии.
- Tanenbaum, гл. 8, §8.4.1 (стр. PDF 852–855).
