---
title: DNS — типы записей
aliases: ["DNS RR types", "A record", "AAAA", "MX", "CNAME", "NS", "TXT", "SOA"]
type: concept
layer: application
chapter: 7
difficulty: basic
prerequisites: ["[[DNS]]"]
related: ["[[DNS — кэширование и TTL]]"]
tags: [networking, ch07]
---
# DNS — типы записей (Resource Records)

## TL;DR
Главные RR-типы: **A** (имя → IPv4), **AAAA** (имя → IPv6), **MX** (mail server), **CNAME** (alias), **NS** (delegate authority), **TXT** (произвольный текст), **SOA** (zone metadata), **PTR** (reverse: IP → имя), **SRV** (service location). Каждая запись имеет TTL.

## Какую проблему решает
DNS — не только «имя → IP». Это общая база, привязывающая разные **типы** информации к именам: SMTP-серверы домена, txt-записи для email-аутентификации (SPF/DKIM/DMARC), сертификаты (CAA), location-сервисы (SRV). Один DNS-запрос с типом возвращает нужное.

## Как работает

| Тип | Назначение | Пример значения |
|---|---|---|
| **A** | IPv4-адрес | 142.250.180.78 |
| **AAAA** | IPv6-адрес | 2607:f8b0:4006:80a::200e |
| **MX** | mail server для домена + priority | `10 mail.example.com.` |
| **CNAME** | alias на другое имя | `www → example.com` |
| **NS** | delegate authority | `ns1.example.com` |
| **TXT** | произвольный текст | SPF, DKIM, verification |
| **SOA** | zone metadata (primary NS, email, serial) | стандартизованная структура |
| **PTR** | IP → имя (reverse DNS) | `78.180.250.142.in-addr.arpa → google.com` |
| **SRV** | service+protocol → host+port | `_sip._udp.example.com → 0 5 5060 sip.example.com` |
| **CAA** | какой CA может выпускать сертификаты | `letsencrypt.org` |
| **DNSKEY/DS/RRSIG** | DNSSEC |  |
| **HTTPS/SVCB** | service binding (HTTP/3 hints, RFC 9460) |  |

**Структура RR:**
```
NAME    TTL   CLASS  TYPE   RDATA
www.example.com. 3600 IN A   142.250.180.78
```

**TXT-записи** — частые применения:
- **SPF:** `v=spf1 include:_spf.google.com ~all`
- **DKIM:** публичный ключ для email-подписи
- **DMARC:** policy для отвергнутых писем
- **Verification:** `google-site-verification=...`

## Пример
```bash
$ dig wikipedia.org

;; ANSWER SECTION:
wikipedia.org.    300  IN  A     208.80.154.224

$ dig wikipedia.org MX
wikipedia.org.    1800 IN  MX    10 mxa-005a8801.gslb.pphosted.com.

$ dig wikipedia.org TXT
wikipedia.org.    1800 IN  TXT   "v=spf1 include:wikimedia.org ~all"
```

## Связи
- **Базируется на:** [[DNS]].
- **Используется в:** mail-системы (MX, SPF, DKIM), CDN (CNAME chains), security (CAA, DNSSEC).
- **Соседи по уровню:** [[DNS — кэширование и TTL]] — каждая запись имеет TTL.
- **Противопоставляется:** —

## Подводные камни
- **CNAME at apex** (для example.com напрямую) был запрещён по стандарту. **ALIAS / ANAME** — vendor-specific обходы; SVCB/HTTPS RR (RFC 9460) — стандартное решение.
- **DNS-rebinding** атака: атакующий быстро меняет IP в A-записи → клиент сначала идёт к нему, потом DNS возвращает victim-IP внутри LAN.
- **Цикл CNAME → CNAME → CNAME** — допустим, но хост проверяет ограничение (~10 уровней) для DoS-защиты.

## Дальше читать
- [[DNS]] — общий контекст.
- [[DNS — кэширование и TTL]] — TTL у каждого RR.
- Tanenbaum, гл. 7, §7.1.4 (стр. PDF 692–699).
