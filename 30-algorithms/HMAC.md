---
title: HMAC
aliases: ["Hash-based Message Authentication Code"]
type: algorithm
layer: security
chapter: 8
difficulty: basic
prerequisites: ["[[Хеш-функции]]"]
related: ["[[Цифровая подпись]]", "[[TLS — рукопожатие]]"]
tags: [networking, ch08]
---
# HMAC — Hash-based MAC (RFC 2104)

## TL;DR
Способ **аутентификации сообщения** с помощью hash + общий секретный ключ. `HMAC(K, M) = H((K ⊕ opad) || H((K ⊕ ipad) || M))`. Получатель пересчитывает HMAC с тем же K — совпало → сообщение от ожидаемого отправителя и не изменено. Защищает от **length extension attack** на naive `H(K||M)`. Используется в TLS, IPsec, JWT, OAuth.

## Какую проблему решает
**Naive MAC:** `H(K || M)` — кажется работает, но уязвим к **length extension attack** на Merkle-Damgård hashes (MD5, SHA-1, SHA-2): зная `H(K||M)` и длину K, атакующий может построить `H(K || M || padding || M2)` без знания K.

HMAC — стандартный safe способ построить MAC из любого hash.

## Как работает

**Формула:**
$$ \text{HMAC}(K, M) = H((K' \oplus opad) \mathbin\Vert H((K' \oplus ipad) \mathbin\Vert M)) $$

где:
- `K'` — ключ, padded до block size hash'а.
- `ipad = 0x36` repeating, `opad = 0x5C` repeating.
- Two passes of H.

**Свойства:**
- Получатель с тем же K пересчитывает HMAC, сравнивает.
- Защита от length extension (двухпроходная конструкция).
- Можно использовать с любым hash — HMAC-SHA-256, HMAC-SHA-512.

## Пример
**TLS 1.2 record protection (исторически):**
- AES-CBC encrypt + HMAC-SHA-256 для каждого record.
- Encrypt-then-MAC.
- В TLS 1.3 заменено на AEAD (GCM, ChaCha20-Poly1305) — HMAC отдельно не нужен.

**JWT (JSON Web Token) with HS256:**
- Header + payload + HMAC-SHA-256(secret, header.payload).
- Signature inserted в token.
- Server verifies on receive.

**API auth (Amazon AWS Sig v4):**
- Каждый запрос подписывается HMAC от компонентов (URL, headers, body).
- Server pересчитывает.

## Связи
- **Базируется на:** [[Хеш-функции]] (любой H).
- **Используется в:** TLS (legacy modes), IPsec (auth), JWT-HS, AWS-Signing, MAC of API tokens.
- **Соседи по уровню:** **Poly1305** — другой MAC, часто paired с ChaCha20 для AEAD; **GMAC** — paired с AES-CTR.
- **Противопоставляется:** [[Цифровая подпись]] (asymmetric) — non-repudiation; HMAC — symmetric, обе стороны знают K.

## Подводные камни
- **Constant-time compare** при verify обязателен — иначе timing attack.
- **HMAC-MD5, HMAC-SHA-1 всё ещё secure** для **MAC** (length-extension не applicable), хотя hash сам сломан для collisions. Но не используй MD5/SHA1 в новых проектах.
- **Key length:** ключ должен быть длиной ≥ output hash для full security.
- **HMAC ≠ digital signature**: обе стороны должны знать K. Атакующий с K может **создать** любой valid HMAC; non-repudiation требует подписи (RSA/ECDSA).

## См. также (прикладное)
RF-circumvention: HMAC — встроен в обфускацию censorship-resistant-протоколов.
- [[MTProxy и FakeTLS]] — HMAC-SHA256 в FakeTLS-handshake (hidden-secret в TLS-random).
- [[telemt]], [[mtg]] — реализации MTProxy с HMAC-проверкой.
- [[VLESS-Reality]] — HKDF (HMAC-based KDF) для деривирования session keys из x25519-ECDH.
- [[applied-rf-status]] — обзор.

## Дальше читать
- [[Хеш-функции]] — base.
- [[Цифровая подпись]] — асимметричная альтернатива.
- Tanenbaum, гл. 8, §8.7.3 (стр. PDF 883–886).
