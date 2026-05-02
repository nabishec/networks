---
title: DES и AES
aliases: ["DES", "AES", "3DES", "Rijndael"]
type: algorithm
layer: security
chapter: 8
difficulty: intermediate
prerequisites: ["[[Симметричная vs асимметричная криптография]]"]
related: ["[[Режимы шифрования]]"]
tags: [networking, ch08]
---
# DES и AES — block ciphers

## TL;DR
**DES** (Data Encryption Standard, IBM/NSA, 1977): 64-битный блок, 56-битный ключ, 16 раундов **Feistel-сети**. Сегодня **сломан** brute-force (1990-е). **3DES** — DES трижды, ~112-бит security. **AES** (Advanced Encryption Standard, 2001, на основе **Rijndael**): 128-битный блок, ключи 128/192/256, 10/12/14 раундов SPN-сети. Современный стандарт, аппаратное ускорение (AES-NI) — гигабиты/секунду.

## Какую проблему решает
Современные сети шифруют **гигабайты данных в секунду**. Нужны быстрые, security-tight симметричные шифры. DES был стандартом 1970-90х; AES заменил его, когда DES стал brute-force-able.

## Как работает

### DES (1977)
- **Feistel-сеть:** разделение блока на L и R; round = L' = R, R' = L ⊕ F(R, K_round).
- 16 раундов с разными round keys (derived from main key).
- **Initial и final permutations** — больше косметика.
- **Слабость:** ключ 56 бит (исходно 64 с parity).
- **Cracked:** 1998 EFF DES Cracker за $250k брутил DES за days.
- **3DES:** Encrypt-Decrypt-Encrypt:
  - **2-ключевой** (K1, K2, K1) → effective ~112 бит security (по Tanenbaum, стр. 868).
  - **3-ключевой** (K1, K2, K3) → теоретически 168 бит, но meet-in-the-middle снижает до ~112.
  - Использовался когда DES был сломан, но AES ещё не стандарт.

### AES (Rijndael, 2001)
- **SPN** (Substitution-Permutation Network), не Feistel.
- 128-битный блок (AES-128/192/256 — длина ключа).
- 10/12/14 раундов соответственно.
- Каждый раунд:
  1. **SubBytes** (S-box на каждый byte).
  2. **ShiftRows** (перестановка байтов).
  3. **MixColumns** (linear combination в GF(2⁸)).
  4. **AddRoundKey** (XOR с round key).
- AES-256 standard для secret-level гос. данных в US.

**Производительность:**
- AES-NI (Intel/AMD) — hardware-инструкции AES → 5-10 GB/s на core.
- Software-only AES — 100-300 MB/s.
- ChaCha20 — соперник AES для устройств без AES-NI (mobile, embedded).

## Пример
- **AES-256-GCM** в TLS 1.3 — main cipher suite.
- **AES-128-CBC** в old TLS 1.2 (с уязвимостями padding-oracle, ушёл).
- **AES в WPA2** (CCMP-mode) — Wi-Fi-шифрование.
- **AES в IPsec** (ESP — encryption) — VPN.

## Связи
- **Базируется на:** теория блочных шифров, GF(2⁸) арифметика.
- **Используется в:** TLS, IPsec, WPA2/3, SSH, full-disk encryption.
- **Соседи по уровню:** [[Режимы шифрования]] (CBC, CTR, GCM на основе block cipher); ChaCha20 (stream cipher альтернатива).
- **Противопоставляется:** stream ciphers (ChaCha20, RC4-устар.) — другая парадигма.

## Подводные камни
- **DES is dead** — не использовать, кроме legacy.
- **3DES** — медленный, deprecated, лучше AES.
- **AES-128 vs AES-256:** для большинства целей AES-128 достаточно (>10²⁰ years brute force). AES-256 — для long-term secrets и quantum-resistance soft-margin.
- **Block cipher needs mode** — сам по себе AES шифрует один блок 128 бит. Для произвольного payload нужен mode (см. [[Режимы шифрования]]).

## Дальше читать
- [[Режимы шифрования]] — как использовать.
- [[RSA]] — асимметричный соседний.
- Tanenbaum, гл. 8, §8.5 (стр. PDF 866–875).
