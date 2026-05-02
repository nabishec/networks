---
title: MIME
aliases: ["Multipurpose Internet Mail Extensions"]
type: concept
layer: application
chapter: 7
difficulty: basic
prerequisites: ["[[Email — архитектура]]"]
related: ["[[SMTP]]", "[[HTTP]]"]
tags: [networking, ch07]
---
# MIME — Multipurpose Internet Mail Extensions (RFC 2045–2049)

## TL;DR
Расширение для email, позволяющее передавать не только ASCII-текст, но и **бинарные файлы (вложения), не-ASCII-текст (UTF-8), HTML, multipart-сообщения**. Кодирование через **base64** или **quoted-printable**. Заголовки `Content-Type`, `Content-Transfer-Encoding`, `Content-Disposition`. Применяется и в HTTP — `Content-Type: text/html; charset=utf-8`.

## Какую проблему решает
SMTP исторически (RFC 821) поддерживал только **7-битный ASCII**. Не-английские символы, бинарные вложения, форматирование — никак. MIME даёт стандартный способ передавать эти **поверх** ASCII-only SMTP.

## Как работает

**Заголовки MIME в email:**
```
MIME-Version: 1.0
Content-Type: multipart/mixed; boundary="----=_Part_123"
```

**Multipart-структура:**
```
------=_Part_123
Content-Type: text/plain; charset=utf-8
Content-Transfer-Encoding: 8bit

Привет, Боб!

------=_Part_123
Content-Type: image/jpeg; name="photo.jpg"
Content-Transfer-Encoding: base64
Content-Disposition: attachment; filename="photo.jpg"

/9j/4AAQSkZJRgABAQAAAQABAAD/2wBDAAEBAQEBAQEB...

------=_Part_123--
```

**Content-Type категории:**
- `text/plain`, `text/html`
- `image/jpeg`, `image/png`, `image/gif`
- `audio/mpeg`, `video/mp4`
- `application/pdf`, `application/json`
- `multipart/mixed`, `multipart/alternative`, `multipart/related`

**Encoding:**
- **7bit:** ASCII as-is.
- **8bit:** non-ASCII bytes (требует ESMTP 8BITMIME).
- **base64:** 3 бинарных байта → 4 ASCII-символа (overhead 33%). Для бинарного.
- **quoted-printable:** для preferentially-ASCII текста с редкими non-ASCII (overhead малый).

**Multipart/alternative:** письмо в HTML и plain text — клиент выберет, что показать.

## Пример
**Письмо с фото:**
- Headers + boundary.
- Часть 1: `text/plain` — «вот фото».
- Часть 2: `image/jpeg` base64-encoded.
- MUA получателя парсит boundary, отображает текст и встраивает картинку.

**HTTP использует тот же подход:**
```
POST /upload HTTP/1.1
Content-Type: multipart/form-data; boundary=----abc

------abc
Content-Disposition: form-data; name="file"; filename="x.jpg"
Content-Type: image/jpeg

<binary data>
------abc--
```

## Связи
- **Базируется на:** [[Email — архитектура]] (его расширение), стандарт RFC.
- **Используется в:** [[SMTP]] (исторически), [[HTTP]] (Content-Type, multipart/form-data, multipart/byteranges), web file uploads.
- **Соседи по уровню:** **S/MIME** — секурное MIME с шифрованием/подписью.
- **Противопоставляется:** raw 7-bit ASCII text — не поддерживает бинари/не-английский.

## Подводные камни
- **base64 = 33% overhead.** Большие attachments в email раздуваются. Иногда лучше уметь 8BITMIME.
- **Content-Type charset** часто опускают → клиент гадает (UTF-8 vs cp1251). Особенно в legacy системах.
- **HTML email** + tracking pixels + remote images — privacy-issues. Современные клиенты блокируют по умолчанию.

## Дальше читать
- [[SMTP]] — главный пользователь.
- [[HTTP]] — повторно использует Content-Type/multipart.
- Tanenbaum, гл. 7, §7.2.3 (стр. PDF 710–716).
