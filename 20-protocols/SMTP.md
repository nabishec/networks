---
title: SMTP
aliases: ["Simple Mail Transfer Protocol", "ESMTP"]
type: protocol
layer: application
chapter: 7
difficulty: basic
prerequisites: ["[[Email — архитектура]]", "[[TCP]]"]
related: ["[[IMAP и POP3]]", "[[MIME]]"]
tags: [networking, ch07]
---
# SMTP — Simple Mail Transfer Protocol (RFC 5321)

## TL;DR
**Протокол передачи почты** между MTA и от MUA к MSA. Поверх TCP. Текстовый, command-response. **Порт 25** (MTA-MTA, без auth), **587** (submission, с auth), **465** (SMTPS, TLS implicit). Старый и простой — главные команды: HELO/EHLO, MAIL FROM, RCPT TO, DATA, QUIT.

## Какую проблему решает
Между почтовыми серверами нужен общий протокол передачи. SMTP появился в 1981 г., оказался настолько простым и достаточным, что не менялся фундаментально 40+ лет. Расширения (ESMTP) добавили auth, TLS, pipelining.

## Как работает

**Базовая SMTP-транзакция:**
```
S: 220 mail.example.com ESMTP ready
C: EHLO sender.example.com
S: 250-mail.example.com Hello sender.example.com
S: 250-AUTH PLAIN LOGIN
S: 250-STARTTLS
S: 250 SIZE 50000000
C: MAIL FROM: <alice@example.com>
S: 250 OK
C: RCPT TO: <bob@gmail.com>
S: 250 OK
C: DATA
S: 354 Send message ending with <CRLF>.<CRLF>
C: From: alice@example.com
C: To: bob@gmail.com
C: Subject: Hello
C:
C: Hi Bob!
C: .
S: 250 OK queued as XYZ123
C: QUIT
S: 221 Bye
```

**Расширения (ESMTP):**
- **STARTTLS:** переключение на TLS поверх existing соединения.
- **AUTH:** PLAIN, LOGIN, CRAM-MD5, OAUTHBEARER.
- **PIPELINING:** отправлять несколько команд без ожидания ответа.
- **8BITMIME:** non-ASCII в теле.
- **DSN:** delivery status notifications.

**Порты:**
- **25** — MTA-MTA (между серверами). Часто блокируется ISP для частных пользователей (анти-спам).
- **587** — submission, с обязательной аутентификацией.
- **465** — SMTPS, TLS implicit (с RFC 8314 — рекомендуется).

## Пример
**Дебаг SMTP:**
```bash
$ openssl s_client -connect smtp.gmail.com:465 -crlf
...
EHLO me.example.com
MAIL FROM:<test@example.com>
...
```

В логах MTA (postfix) видно полный SMTP-обмен.

## Связи
- **Базируется на:** [[Email — архитектура]] (его роль), [[TCP]] (transport).
- **Используется в:** Postfix, Sendmail, Exim — все Linux MTA; Microsoft Exchange.
- **Соседи по уровню:** [[IMAP и POP3]] — обратная сторона (забор).
- **Противопоставляется:** Microsoft proprietary EAS, Exchange Web Services — закрытые альтернативы.

## Подводные камни
- **Plain text по умолчанию** — без STARTTLS читается провайдером. Современные MTA настоятельно требуют TLS.
- **MTA-STS, DANE** — современные mechanism гарантирующие, что **mail-серверы шифруются** при передаче.
- **8-битные тела** не поддерживались в RFC 821; MIME (см. [[MIME]]) обходил через base64. ESMTP 8BITMIME решает напрямую.

## Дальше читать
- [[Email — архитектура]] — где SMTP в стэке.
- [[IMAP и POP3]] — для приёма.
- Tanenbaum, гл. 7, §7.2.4 (стр. PDF 716–721).
