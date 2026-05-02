---
title: Эллиптическая криптография ECC
aliases: ["ECC", "ECDSA", "ECDH", "Ed25519", "Curve25519"]
type: algorithm
layer: security
chapter: 8
difficulty: advanced
prerequisites: ["[[RSA]]", "[[Диффи-Хеллман]]"]
related: ["[[TLS — рукопожатие]]", "[[Цифровая подпись]]"]
tags: [networking, ch08]
---
# Эллиптическая криптография (ECC)

## TL;DR
Альтернатива RSA на основе **дискретного логарифма в группе точек эллиптической кривой**. **Главное преимущество:** **меньшие ключи** при той же security — ECC-256 ≈ RSA-3072 ≈ 128-bit security. Главные алгоритмы: **ECDH** (key exchange — заменяет DH), **ECDSA** (signatures — заменяет RSA-sign), **Ed25519** (modern signature, очень быстрый). В TLS 1.3 — default key exchange (ECDHE).

## Какую проблему решает
RSA-2048 даёт ~112 бит security, ключи ~256 байт. На mobile/IoT/blockchain это много: bandwidth, storage, CPU. ECC даёт ту же security при ключах **в 10× меньше** → лучше для constrained devices, blockchain (Bitcoin использует secp256k1).

Также: ECC **быстрее** RSA по операциям, особенно signing.

## Как работает

**Эллиптическая кривая над GF(p):**
$$ y^2 = x^3 + ax + b \pmod p $$

Точки на кривой образуют **группу** с операцией «сложение» (геометрическая):
- P + Q: провести прямую через P,Q → пересечение с кривой → отразить → R.
- 2P (удвоение): касательная в P → отразить.

**ECDLP** (Elliptic Curve Discrete Log Problem): зная G и P=kG, найти k — **сложно** (best-known O(√n)).

**ECDH:**
1. Public params: кривая + generator G.
2. Alice: random a → A = aG, шлёт.
3. Bob: random b → B = bG, шлёт.
4. Alice: aB = abG; Bob: bA = abG → общий секрет.

**ECDSA** (signatures): использует ECDLP-сложность для подписи + verify; формула почти как DSA, но в группе ECC.

**Curves:**
- **P-256 / secp256r1** (NIST) — стандарт, в TLS-сертификатах.
- **secp256k1** — Bitcoin (по историческим причинам).
- **Curve25519** (Bernstein, 2005) — современный design, immune to timing attacks. Используется в Signal, WireGuard, TLS 1.3.
- **Ed25519** — Edwards-form Curve25519 для signatures, очень быстрый.

**Размеры ключей** (одинаковая security):

| Security level | RSA | ECC |
|---|---|---|
| 80-bit | 1024 | 160 |
| 112-bit | 2048 | 224 |
| 128-bit | 3072 | 256 |
| 192-bit | 7680 | 384 |
| 256-bit | 15360 | 521 |

## Пример
**TLS 1.3 default:**
- ClientHello предлагает X25519 (Curve25519 для DH).
- Server выбирает X25519.
- ECDHE-handshake — 32-байтные public keys (vs ~300 у RSA).
- Speed: 1 ms vs 10 ms у RSA-2048.

**SSH key:** `ssh-keygen -t ed25519` создаёт 32-байтный private + 32-байтный public. Vs `-t rsa` — 256+256 байт.

**Bitcoin transactions** подписаны secp256k1 ECDSA. Без ECC blockchain bloated был бы значительно больше.

## Связи
- **Базируется на:** теория конечных полей, [[Диффи-Хеллман]] (концептуально), [[Цифровая подпись]] (применение).
- **Используется в:** [[TLS — рукопожатие]] (ECDHE default), SSH (Ed25519), WireGuard (Curve25519), Bitcoin (secp256k1), Apple iMessage, Signal.
- **Соседи по уровню:** **Pairing-based crypto** — расширение ECC; **post-quantum signatures** (ML-DSA, SLH-DSA).
- **Противопоставляется:** [[RSA]] — большие ключи, медленнее, но проще понимается математически.

## Подводные камни
- **Curve choice critical:** некоторые NIST-кривые (P-256) подозреваются в backdoor (Dual_EC_DRBG история). Современный совет — Curve25519/Ed25519.
- **Side-channel attacks:** наивные ECC-implementations уязвимы к timing/cache атакам. Constant-time Curve25519 спроектирован для устойчивости.
- **Quantum threat:** ECDLP сломаем Shor'ом (как и RSA). Надо мигрировать на post-quantum.
- **secp256k1 ≠ secp256r1** — разные кривые! Bitcoin использует k1 (специфический), TLS — r1 (NIST).

## Дальше читать
- [[RSA]] — для контраста.
- [[Диффи-Хеллман]] — родственник по идее.
- Tanenbaum, гл. 8, §8.6.2 (стр. PDF 878–879).
