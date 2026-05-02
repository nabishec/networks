---
title: DNS over HTTPS / TLS
aliases: ["DoH", "DoT", "DNS over HTTPS", "DNS over TLS", "DNS over HTTPS / TLS", "DNS over HTTPS - TLS"]
type: protocol
layer: application
chapter: 7
difficulty: intermediate
prerequisites: ["[[DNS]]", "[[HTTPS]]", "[[TLS — рукопожатие]]"]
related: ["[[DNS — приватность и ODNS]]", "[[DNS spoofing]]", "[[Encrypted DNS — DoH-DoT]]"]
tags: [networking, ch07]
---
# DNS over HTTPS / TLS — DoH и DoT

## TL;DR
Два независимых способа **зашифровать DNS-запросы**, чтобы провайдер/сторонний наблюдатель не видел, какие домены вы резолвите, и не мог подменить ответ.
- **DoT** (RFC 7858, 2016) — DNS-сообщение в обычном wire-формате внутри TLS-канала на порту **853** TCP. Прозрачно для DNS-инструментов; виден провайдеру как «трафик к DNS», но содержание зашифровано.
- **DoH** (RFC 8484, 2018) — DNS-сообщение поверх HTTPS на порту **443**. Снаружи неотличим от обычного веб-трафика — труднее заблокировать, но усложняет управление сетью.

В книге Tanenbaum 6e приватность DNS обсуждается в §7.1.7 без подробной разбивки DoH vs DoT — детали ниже собраны из статей на Хабре (см. «Источники»).

## Какую проблему решает
Классический DNS на UDP/53 — **открытый текст**:
- провайдер видит каждый домен, который вы запрашиваете → строит профиль;
- любой узел между клиентом и resolver'ом может **подменить ответ** ([[DNS spoofing]]) — основа цензуры в РФ и многих странах;
- DNSSEC защищает от подмены, но не от подсматривания.

Шифрование самого DNS-канала закрывает оба фронта.

## Как работает

### DoT (DNS-over-TLS, RFC 7858)
1. Клиент открывает TCP-соединение к resolver'у на порт **853**.
2. Поверх TCP — TLS handshake (сертификат resolver'а, обычно публичный CA).
3. Внутри TLS — обычный DNS-протокол (тот же binary wire-format, что и UDP/53).
4. Возможен strict authentication: клиент сверяет SPKI-pin сервера.

Снаружи виден факт TLS-соединения на 853 → провайдер **знает, что это DoT**, но не содержание.

### DoH (DNS-over-HTTPS, RFC 8484)
1. Клиент открывает HTTPS-сессию к URL вида `https://1.1.1.1/dns-query` или `https://dns.google/dns-query`.
2. DNS-запрос упаковывается одним из двух способов:
   - **GET:** `?dns=<base64url(dns_message)>` (wire format в URL).
   - **POST:** тело — `application/dns-message` с raw-байтами.
3. Ответ — `Content-Type: application/dns-message` с тем же wire-format.
4. Поверх HTTPS обычно работает HTTP/2 → мультиплексирование запросов.

Существует **JSON-API** (Google `https://dns.google/resolve?name=...&type=A`), но это **не часть** RFC 8484 — это удобство для веба, не «канонический DoH» (по [habr.com/ru/articles/898138](https://habr.com/ru/articles/898138/)).

### Сравнение
| | DoT | DoH |
|---|---|---|
| RFC | 7858 (2016) | 8484 (2018) |
| Порт | TCP **853** | TCP **443** (HTTPS) |
| Транспорт | TLS | HTTPS (TLS + HTTP/2) |
| Формат | DNS wire | DNS wire поверх HTTP-body |
| Видимость для провайдера | «это DNS» (по порту), содержание скрыто | неотличимо от обычного HTTPS |
| Управление в сети | блокировать/контролировать просто (порт 853) | сложно — придётся ломать весь HTTPS |
| Поддержка ОС | в Android Pie+ — by default | в браузерах (Firefox, Chrome), в iOS 14+, в Windows 11 |
| Простота для приложения | TLS-сокет → DNS поверх | HTTPS-клиент достаточен |

## Пример

### DoT через `systemd-resolved` (Linux)
```ini
# /etc/systemd/resolved.conf
[Resolve]
DNS=1.1.1.1#one.one.one.one 9.9.9.9#dns.quad9.net
DNSOverTLS=yes
```
`systemd-resolved` устанавливает TLS на 853, проверяет сертификат `one.one.one.one`, шлёт DNS внутри.

### DoT через `dig` (новые версии 9.18+)
```bash
dig +tls @1.1.1.1 example.com
```

### DoH в Firefox
`about:preferences#privacy` → **Enable DNS over HTTPS** → `Cloudflare`/`NextDNS`/`Custom`.

### DoH в браузерных приложениях (curl 7.62+)
```bash
curl --doh-url https://dns.google/dns-query https://example.com
```

## Связи
- **Базируется на:** [[DNS]] (содержание), [[TLS — рукопожатие]] (канал в DoT), [[HTTPS]] (канал в DoH), HTTP/2 (мультиплексирование запросов).
- **Используется в:** [[DNS — приватность и ODNS]] (как первый шаг к приватности), современные браузеры и ОС, censorship-circumvention в РФ (см. [[Encrypted DNS — DoH-DoT]]).
- **Соседи по уровню:** **DNSCrypt** (старший pre-RFC аналог от OpenDNS), **DNS-over-QUIC** (RFC 9250, на UDP/443) — современная альтернатива.
- **Противопоставляется:** plain DNS (UDP/53) — открытый, уязвим к [[DNS spoofing]] и слежке.

## Подводные камни
- **Bootstrap-проблема:** чтобы открыть TLS к `1.1.1.1` или `dns.google`, нужно знать IP. Если только domain — fallback на обычный DNS, что подрывает приватность. Решение — **bootstrap IP** в конфиге.
- **Trust transfer:** DNS теперь видит **resolver-провайдер** (Cloudflare, Google) вместо ISP. Это перенос доверия, не уничтожение.
- **Корпоративные сети:** DoH в браузерах **обходит системные DNS-настройки** → ломает enterprise-фильтрацию, malware-blocklist, parental control. Многие корпоративные ИТ-департаменты поэтому отключают DoH централизованно.
- **DoH ≠ универсальное решение от цензуры:** даже зашифровав DNS, сертификат сайта (SNI) виден в plain text при HTTPS-handshake → провайдер по-прежнему узнаёт destination. Полное решение — DoH + ECH (Encrypted ClientHello).
- **Критика DoH** (см. [habr.com/ru/post/473756](https://habr.com/ru/post/473756/)): автор считает, что DoH «маскирует DNS под обычный HTTPS» — это архитектурно усложняет администрирование сетей и встроенных систем. По его мнению, DoT — более чистое решение.
- **РФ-2025+:** Yandex DNS (77.88.8.8) поддерживает DoH/DoT и часто остаётся доступен из мобильных whitelist-сетей; Cloudflare (1.1.1.1) и Google (8.8.8.8) — периодически блокируются. Подробнее в [[Encrypted DNS — DoH-DoT]].

## Источники / Дальше читать
- [[DNS]] — базовый протокол.
- [[DNS — приватность и ODNS]] — следующий шаг (двойной hop, ODoH).
- [[Encrypted DNS — DoH-DoT]] — applied-RF-сторона.
- Habr: [Как HTTP(S) используется для DNS: DoH на практике](https://habr.com/ru/articles/898138/) — Apr 2025.
- Habr: [Как DNS работает через TLS: DoT на практике](https://habr.com/ru/articles/891374/) — Mar 2025.
- Habr: [DNS по HTTPS — половинчатое и неверное решение](https://habr.com/ru/post/473756/) — критическое сравнение DoH vs DoT.
- Habr: [DNS Over TLS & Over HTTPS теперь и на iOS/Android](https://habr.com/ru/articles/429648/) — обзор массового deployment'а.
- Habr: [Минимизация рисков использования DoH/DoT](https://habr.com/ru/articles/523676/) — администрирование сети.
- Habr: [Что происходит с протоколом DNS-over-HTTPS](https://habr.com/ru/companies/vasexperts/articles/723210/) — состояние на 2023.
- Tanenbaum, гл. 7, §7.1.7 — общая privacy-тема DNS, отдельных глав DoH/DoT в книге нет.
