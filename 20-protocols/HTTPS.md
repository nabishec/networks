---
title: HTTPS
aliases: ["HTTP Secure", "HTTP over TLS"]
type: protocol
layer: application
chapter: 7
difficulty: basic
prerequisites: ["[[HTTP]]", "[[TLS — рукопожатие]]"]
related: ["[[X.509 сертификаты]]", "[[HTTP/2 и HTTP/3]]"]
tags: [networking, ch07]
---
# HTTPS — HTTP over TLS

## TL;DR
**HTTP** поверх **TLS** (Transport Layer Security). Шифрует трафик, аутентифицирует сервер через **X.509-сертификат**, защищает от MITM. Порт 443. С 2018 г. — стандарт de facto для всех публичных сайтов; HTTP без S браузеры маркируют «Not Secure». TLS 1.3 (RFC 8446) — текущая основа.

## Какую проблему решает
HTTP — открытый текст: любой между клиентом и сервером (провайдер, Wi-Fi-злоумышленник, государство) видит и может изменить. Для современного веба (банки, soc.networks, mail) это неприемлемо. HTTPS даёт три гарантии:
- **Confidentiality:** данные зашифрованы.
- **Integrity:** изменения детектируются.
- **Authentication:** сервер тот, за кого себя выдаёт (через сертификат).

## Как работает

**Структура соединения:**
```
+-----------+
|   HTTP    |
+-----------+
|   TLS     |  ← шифрование, аутентификация
+-----------+
|   TCP     |
+-----------+
```

**TLS handshake** (см. [[TLS — рукопожатие]]):
1. ClientHello — список cipher suites, ALPN ("h2", "http/1.1"), SNI (server name indication).
2. ServerHello — выбранный cipher; сертификат; (TLS 1.3) ключи.
3. Обмен ключами через Diffie-Hellman.
4. Finished — handshake закончен, начинается шифрованный поток.

**TLS 1.3:** 1 RTT для нового соединения, 0-RTT для resumed.

**Цепочка сертификатов:**
- Сервер: example.com.
- Сертификат подписан CA (например, Let's Encrypt R3).
- R3 подписан root (ISRG Root X1) — root в браузере trusted.
- Браузер проверяет цепочку: R3 → ISRG Root → match.

**SNI** (Server Name Indication): в первом TLS-фрагменте клиент сообщает, к какому **virtual host** обращается → сервер выбирает нужный сертификат. Без SNI один IP мог бы обслуживать только один HTTPS-сайт.

## Пример
**Открытие `https://wikipedia.org`:**
1. DNS → IP.
2. TCP connect 443.
3. TLS handshake: SNI=wikipedia.org, ALPN=h2.
4. Сервер: сертификат wikipedia.org issued by Let's Encrypt.
5. Браузер проверяет цепочку — OK.
6. Шифрованный канал.
7. HTTP/2-запросы, мультиплексирование.

**Проверка сертификата:**
```bash
$ openssl s_client -connect wikipedia.org:443 -servername wikipedia.org
```

## Связи
- **Базируется на:** [[HTTP]] (приложение), [[TLS — рукопожатие]] (security), [[X.509 сертификаты]] (аутентификация).
- **Используется в:** **весь** современный публичный веб; API; OAuth-flows.
- **Соседи по уровню:** [[HTTP/2 и HTTP/3]] (поверх TLS); QUIC (TLS встроенный).
- **Противопоставляется:** plain HTTP — устарел для public web.

## Подводные камни
- **Mixed content:** HTTPS-страница с HTTP-ресурсами → блокировка.
- **Self-signed certs** дают warning. Real CA обязателен в публике.
- **HSTS** (HTTP Strict Transport Security): сервер шлёт header → браузер запоминает «всегда HTTPS для этого домена», блокирует HTTP-fallback.
- **Certificate Transparency** (CT): все выпущенные сертификаты публикуются в публичных логах — детектирование misissue.
- **Let's Encrypt** (2015) сделал сертификаты бесплатными и автоматизированными → массовая HTTPS-адаптация.

## Дальше читать
- [[TLS — рукопожатие]] — детали.
- [[X.509 сертификаты]] — PKI.
- Tanenbaum, гл. 7, §7.3.4 (стр. PDF 740–753); гл. 8, §8.12.3.
