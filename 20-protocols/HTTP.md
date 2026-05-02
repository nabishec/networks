---
title: HTTP
aliases: ["Hypertext Transfer Protocol", "HTTP/1.1"]
type: protocol
layer: application
chapter: 7
difficulty: basic
prerequisites: ["[[Web — архитектура]]", "[[TCP]]"]
related: ["[[HTTP-2 и HTTP-3|HTTP/2 и HTTP/3]]", "[[HTTPS]]", "[[Cookies и web tracking]]"]
tags: [networking, ch07]
---
# HTTP — HyperText Transfer Protocol (RFC 9110, 9112)

## TL;DR
Протокол прикладного уровня для веба. Текстовый, request-response, **stateless** (по умолчанию). Запрос: метод + URL + headers + (опц. body). Ответ: status + headers + body. **HTTP/1.1** (1997) — текст; **HTTP/2** (2015) — бинарный, multiplexed; **HTTP/3** (2022) — поверх QUIC. Базис веба, REST API, многих RPC-систем.

## Какую проблему решает
WWW нужен простой протокол **запрос-ответ** между браузером и сервером. HTTP — текстовый, легко отлаживать (telnet → GET), легко расширять через headers. Семантика осталась стабильной с 1991 г., транспорт менялся (TCP → TCP+TLS → QUIC).

## Как работает

**Request:**
```
GET /index.html HTTP/1.1
Host: example.com
User-Agent: curl/8.0
Accept: text/html
```

**Response:**
```
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 1234
Cache-Control: max-age=3600

<html>...</html>
```

**Методы:**
- **GET** — получить ресурс. Идемпотентный, кэшируемый.
- **POST** — создать. Не идемпотентный.
- **PUT** — заменить. Идемпотентный.
- **DELETE** — удалить. Идемпотентный.
- **PATCH** — частично обновить.
- **HEAD** — как GET, но только заголовки.
- **OPTIONS** — какие методы поддерживаются.

**Status-коды:**
- **1xx** Informational (100 Continue, 101 Switching Protocols).
- **2xx** Success (200 OK, 201 Created, 204 No Content).
- **3xx** Redirection (301 Permanent, 302 Found, 304 Not Modified).
- **4xx** Client Error (400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 429 Too Many Requests).
- **5xx** Server Error (500 Internal, 502 Bad Gateway, 503 Service Unavailable).

**Стандартные headers:**
- **Host** — virtual hosting.
- **Content-Type, Content-Length** — что в body.
- **Cache-Control, ETag, If-None-Match** — кэширование.
- **Cookie, Set-Cookie** — состояние.
- **Authorization** — auth.
- **User-Agent** — кто клиент.
- **Accept** — что клиент готов принять.

**Persistent connections (keep-alive):** одно TCP-соединение для многих запросов.

**Pipelining:** отправлять следующий запрос не дождавшись ответа на предыдущий. В HTTP/1.1 redko используется (HoL); HTTP/2 решил properly.

## Пример
```bash
$ curl -v https://wikipedia.org/wiki/Main_Page
> GET /wiki/Main_Page HTTP/1.1
> Host: wikipedia.org
> Accept: */*
< HTTP/1.1 301 Moved Permanently
< Location: https://en.wikipedia.org/wiki/Main_Page
```

## Связи
- **Базируется на:** [[Web — архитектура]], [[TCP]] (HTTP/1.1, /2), [[QUIC]] (HTTP/3).
- **Используется в:** все веб-приложения, REST API, gRPC (поверх HTTP/2), webhooks, OAuth.
- **Соседи по уровню:** [[HTTPS]] (TLS), [[HTTP-2 и HTTP-3|HTTP/2 и HTTP/3]].
- **Противопоставляется:** [[WebSocket]] — bidirectional поверх HTTP-upgrade; [[gRPC]] — структурный RPC.

## Подводные камни
- **Stateless по умолчанию** → cookies/токены для состояния.
- **Caching headers** — частая ошибка: либо нет, либо ломаются после deploy. Etag + Cache-Control правильная пара.
- **CORS** — security policy для cross-origin запросов в браузерах.
- **HTTP/1.1 HoL blocking** на pipelining — главная мотивация HTTP/2.

## Дальше читать
- [[HTTPS]] — поверх TLS.
- [[HTTP-2 и HTTP-3|HTTP/2 и HTTP/3]] — современные эволюции.
- [[Cookies и web tracking]] — состояние.
- Tanenbaum, гл. 7, §7.3.4 (стр. PDF 740–753).
