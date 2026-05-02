---
title: DNS
aliases: ["Domain Name System"]
type: protocol
layer: application
chapter: 7
difficulty: basic
prerequisites: ["[[UDP]]", "[[TCP]]"]
related: ["[[DNS — типы записей]]", "[[DNS — рекурсивный vs итеративный запрос]]", "[[DNS — кэширование и TTL]]", "[[DNS over HTTPS - TLS|DNS over HTTPS / TLS]]"]
tags: [networking, ch07]
---
# DNS — Domain Name System (RFC 1034, 1035)

## TL;DR
**Распределённая иерархическая база** имён → IP. Преобразует `wikipedia.org` → `208.80.154.224`. Запрос обычно идёт на UDP/53, ответ — UDP/53; для больших ответов TCP/53. Иерархия: **корневые серверы** (`.`) → **TLD** (`.org`) → **authoritative** (для `wikipedia.org`). Без DNS интернет невозможно использовать практически.

## Какую проблему решает
Люди запоминают `google.com`, не `142.250.180.78`. IP-адреса меняются, серверов на одно имя могут быть десятки. DNS — слой абстракции между **именами** и **адресами**, плюс инструмент управления (легко добавить/убрать backend, не меняя имени).

## Как работает

**Иерархия имён** — справа налево:
```
www.example.com.
└─┬─┘ └──┬──┘ └┬┘ └─ root (".")
  │     │     └── TLD (.com)
  │     └─── second-level domain (example.com)
  └─── subdomain (www)
```

**Сетевая иерархия серверов:**
1. **Root servers** (a.root-servers.net … m.root-servers.net) — 13 логических, через [[Anycast]] физически тысячи. Знают TLD-серверы.
2. **TLD servers** (`.com`, `.org`, `.ru` …). Знают authoritative для domains.
3. **Authoritative servers** для конкретного домена. Хранят реальные записи.

**Резолверы** (resolver):
- **Stub** — на клиенте, шлёт запрос на recursive.
- **Recursive** — у провайдера или общественные (1.1.1.1, 8.8.8.8). Делают всю работу за клиента.

**Минимальный обмен:**
- Клиент шлёт `8.8.8.8:53 — query for wikipedia.org A`.
- Recursive (через iterations с root → TLD → authoritative) находит ответ.
- Клиент получает `A 208.80.154.224, TTL 300`.

## Пример
**Открыть `wikipedia.org` впервые:**
1. Браузер → ОС → recursive resolver (8.8.8.8 в /etc/resolv.conf).
2. Resolver: «у меня нет в кэше; идём итеративно».
3. Запрос корневому: «кто отвечает за .org?» → ответ: NS-серверы для .org.
4. Запрос .org NS: «кто отвечает за wikipedia.org?» → NS-серверы Wikipedia.
5. Запрос auth: «A для wikipedia.org?» → IP.
6. Resolver кэширует, возвращает клиенту.

## Связи
- **Базируется на:** [[UDP]] (default), [[TCP]] (большие ответы).
- **Используется в:** буквально всё в интернете требует DNS; [[Anycast]] для root/popular DNS; [[CDN — устройство]] (DNS-based routing).
- **Соседи по уровню:** [[DNS — типы записей]], [[DNS — рекурсивный vs итеративный запрос]], [[DNS — кэширование и TTL]], [[DNS over HTTPS - TLS|DNS over HTTPS / TLS]] — детализация.
- **Противопоставляется:** **/etc/hosts** статическая запись — не масштабируется.

## Подводные камни
- DNS-запрос **в открытом виде** (UDP/53) — провайдер видит всё. Защита — DoH/DoT.
- **DNS-spoofing / cache poisoning** — атаки на кэши. Защита — DNSSEC.
- Кэширование с большим TTL означает: после смены IP часть мира продолжает «звонить» по старому. Уменьшать TTL перед миграцией.

## См. также (прикладное)
RF-circumvention: DNS — критическая точка входа для blocked-сайтов.
- [[Encrypted DNS — DoH-DoT]] — DoH/DoT в РФ (Yandex DoH разрешён, Cloudflare часто блокирован).
- [[DNS-туннелирование]] — last-resort tunnel поверх DNS-запросов (~1 KB/s).
- [[ECH и ESNI]] — связано с DNS HTTPS RR (RFC 9460); ESNI **сломан** в РФ.
- [[DPI-фильтрация в РФ]] — DNS-spoofing на провайдер-уровне.
- [[applied-rf-status]] — обзор.

## Дальше читать
- [[DNS — типы записей]], [[DNS — рекурсивный vs итеративный запрос]], [[DNS — кэширование и TTL]] — детальнее.
- [[DNS over HTTPS - TLS|DNS over HTTPS / TLS]] — современный privacy.
- Tanenbaum, гл. 7, §7.1 (стр. PDF 684–704).
