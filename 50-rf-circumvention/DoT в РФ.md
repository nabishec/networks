---
title: DoT в РФ
aliases: ["DoT в обходе", "DNS over TLS в РФ", "DoT"]
type: technique
layer: application
status: broken
status_as_of: 2026-05-02
risk: low
prerequisites: ["[[DNS — приватность и ODNS]]", "[[TLS — рукопожатие]]"]
related: ["[[Encrypted DNS — DoH-DoT]]", "[[DoH в РФ]]"]
sources: [src-05, src-01]
tags: [networking, rf-circumvention, technique]
---
# DoT в РФ — DNS over TLS

## TL;DR
**DoT** (RFC 7858) шифрует DNS внутри **TLS на отдельном порту 853**. Запрос/ответ — обычный wire-format DNS, но в TLS-туннеле. В РФ-2026 — **в большинстве случаев broken**: трафик на TCP/853 узнаваем по самому факту порта и **дропается на мобильных операторах** ещё до TLS-handshake. На домашних провайдерах — **inconsistent**: иногда проходит, чаще нет. Для обхода РФ-2026 [[DoH в РФ]] предпочтительнее DoT именно потому, что DoH «прячется» в HTTPS на 443.

## Какую проблему решает
Та же что у [[DoH в РФ]]: скрыть DNS-запросы от провайдера и убрать DNS-step из SNI-фильтрации. Но реализация другая — отдельный порт 853.

## Как работает
1. Клиент устанавливает TLS-соединение на `port 853` к DoT-resolver'у (например, `1.1.1.1:853`, `8.8.8.8:853`, `dns.yandex.ru:853`).
2. Внутри TLS — стандартное DNS-сообщение (как над port 53, но в шифрованном канале).
3. Сервер аутентифицирует себя сертификатом (PKIX или DANE).

## Где ломается / почему может не работать
- **Маркер по порту:** TCP/853 = «здесь точно DoT». Мобильные операторы РФ блокируют 853 на L4 списком (src-05).
- **Whitelist-модель:** в default-deny не-443 порты к небелым IP не проходят вовсе. DoT — non-443 → блокирован «бесплатно».
- **Узкая поддержка:** клиентов с DoT-only гораздо меньше, чем с DoH; в браузерах DoT отсутствует.
- **NAT/firewall на корпоративных сетях:** часто разрешают только 443 → DoT не пройдёт даже без РКН.
- **DoT vs DoH в РФ — почти всегда не в пользу DoT:** DoH иногда проходит до Yandex; DoT почти всегда блокирован у мобильных операторов и часто — у домашних.

## Минимальный пошаговый сценарий

**Linux (systemd-resolved, DoT к Yandex — единственный шанс пройти):**
```ini
# /etc/systemd/resolved.conf
[Resolve]
DNS=77.88.8.8#common.dns.yandex.net
DNSOverTLS=yes
```

**Android Private DNS** (Settings → Network → Private DNS):
- Hostname: `common.dns.yandex.net` (Yandex) — может работать.
- Hostname: `1dot1dot1dot1.cloudflare-dns.com` — обычно блокируется на мобильных.

**Проверка работы:**
```bash
kdig -d @1.1.1.1 +tls habr.com   # требует knot-utils
# Если timeout / connection refused — порт 853 дропается.
```

**Свой DoT-сервер** (Unbound с tls-port 853): возможно, но **бессмысленно для обхода в РФ** — порт 853 на VPS будет блокирован у клиента так же, как и на public-resolver. Проще поднять DoH на 443.

## Что нужно
- ОС/клиент с поддержкой DoT (Android 9+, systemd-resolved, Unbound, dnsdist).
- DoT-resolver, доступный по 853 (Yandex 77.88.8.8 — самый надёжный в РФ).
- Открытый исходящий TCP/853 (чаще всего у мобильных РФ — закрыт).

## Связи
- **Базируется на:** [[DNS — приватность и ODNS]], [[TLS — рукопожатие]].
- **Парный с:** [[DoH в РФ]] — DoH прячется на 443, DoT светится на 853 → DoH в РФ предпочтителен.
- **Сравнение в:** [[Encrypted DNS — DoH-DoT]].
- **Противопоставляется:** plain DNS (порт 53) — тоже легко блокируем, но иначе.

## Источники
- src-05 — мобильные операторы РФ дропают non-443 → DoT отваливается.
- src-01 — Encrypted DNS как один из шести способов (упоминание).
- [Как DNS работает через TLS: DNS-over-TLS на практике — habr 891374](https://habr.com/ru/articles/891374/)
- [Минимизация рисков использования DNS-over-TLS (DoT) и DNS-over-HTTPS (DoH) — habr 523676](https://habr.com/ru/articles/523676/)
