---
title: PGP и S/MIME
aliases: ["PGP и S/MIME", "PGP", "OpenPGP", "GPG", "S/MIME"]
type: protocol
layer: security
chapter: 8
difficulty: intermediate
prerequisites: ["[[Симметричная vs асимметричная криптография]]", "[[X.509 сертификаты]]"]
related: ["[[Email — архитектура]]"]
tags: [networking, ch08]
---
# PGP и S/MIME — end-to-end email security

## TL;DR
Два конкурирующих стандарта **end-to-end шифрования** и **подписи** для email. **PGP** (Pretty Good Privacy, Phil Zimmermann 1991, OpenPGP RFC 4880): **web of trust** — пользователи лично подписывают ключи друг друга. **S/MIME** (RFC 8551): на основе **X.509-сертификатов** от CA — same trust model как HTTPS. Используется реже массово (UX-проблемы), но критично в гос. и крипто-сообществах.

## Какую проблему решает
SMTP (с STARTTLS) шифрует **между серверами**, но конечный mailbox видит open content. Любой админ почтового сервера, любой государственный запрос — читают. **End-to-end**: только sender и receiver видят open text — серверы посередине видят только зашифрованное.

## Как работает

**Гибридная схема (одинаковая в PGP и S/MIME):**
1. Sender генерирует случайный **session key**.
2. Шифрует сообщение **симметрично** (AES) сессионным ключом.
3. Шифрует **session key асимметрично** (RSA) **public key получателя**.
4. Шлёт зашифрованное сообщение + зашифрованный session key.
5. Receiver: decrypt session key с своим **private** → AES-decrypt тело.

**Подпись:**
- Hash сообщения → подпись с private (sender) → шлёт вместе с письмом.
- Receiver: verify с public (sender) → integrity + authenticity.

### PGP
- **Web of trust:** пользователи подписывают ключи людей, которым лично доверяют. Цепочки доверия — без CA.
- **Key servers:** SKS, keys.openpgp.org, hkps.pool.sks-keyservers.net.
- **OpenPGP** (RFC 4880) — стандарт; **GnuPG (GPG)** — главная open-source реализация.
- Используется в Linux distros (signed packages), security communities.

### S/MIME
- На X.509-сертификатах ([[X.509 сертификаты]]).
- CA-based: trust через PKI.
- Native в Outlook, Apple Mail.
- Enterprise: ваша компания CA выдаёт корп. cert каждому сотруднику.

## Пример
**PGP encrypt:**
```bash
$ gpg --encrypt --recipient bob@example.com message.txt
$ # → message.txt.gpg
$ mail bob@example.com < message.txt.gpg
```

**Receiver:**
```bash
$ gpg --decrypt message.txt.gpg
```

**S/MIME в Outlook:** автоматически — клик «Encrypt», Outlook берёт public cert получателя из directory, шифрует.

## Связи
- **Базируется на:** [[Симметричная vs асимметричная криптография]] (гибрид), [[Цифровая подпись]], [[X.509 сертификаты]] (для S/MIME).
- **Используется в:** [[Email — архитектура]] — поверх SMTP/IMAP.
- **Соседи по уровню:** **Signal** (different protocol, modern E2E for chat); **Matrix** olm/megolm; **Enigmail/Mailvelope** — PGP-плагины для веб-mail.
- **Противопоставляется:** plain SMTP с STARTTLS — без E2E, читается на сервере.

## Подводные камни
- **UX disaster:** key management сложно для нетехнических. Поэтому E2E email никогда не стал массовым.
- **Metadata not encrypted:** Subject, From, To, Cc, headers — open. Атакующий видит граф communicaiton.
- **Forward secrecy nope:** если private key ever compromised → все past messages decrypted. Современные messengers (Signal) — каждый message со свежим ключом.
- **Lost private key = lost mail.** Backup критичен.
- **PGP keyservers** известны как "junk" — заваливаются ключами, нельзя удалить опубликованный.

## Дальше читать
- [[Email — архитектура]] — где эти protocols работают.
- [[X.509 сертификаты]] — для S/MIME.
- Tanenbaum, гл. 8, §8.11 (стр. PDF 921–926).
