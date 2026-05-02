---
title: Encrypted DNS (DoH/DoT) — сравнение в обходе
aliases: ["Encrypted DNS", "Encrypted DNS — DoH-DoT", "DoH-DoT-сводка"]
type: technique
layer: application
status: partial
status_as_of: 2026-05-02
risk: low
prerequisites: ["[[DNS — приватность и ODNS]]", "[[HTTPS]]", "[[TLS — рукопожатие]]"]
related: ["[[DoH в РФ]]", "[[DoT в РФ]]", "[[VLESS-Reality]]", "[[ECH и ESNI]]"]
sources: [src-06, src-01, src-05]
tags: [networking, rf-circumvention, technique]
---
# Encrypted DNS (DoH/DoT) — сравнение в обходе РФ

> **Это сводная заметка.** Атомные детали — в [[DoH в РФ]] и [[DoT в РФ]].
> Теоретическая база протоколов — в [[40-glossary/DNS over HTTPS - TLS]].

## TL;DR
**DoH** (DNS over HTTPS, RFC 8484, **порт 443**) и **DoT** (DNS over TLS, RFC 7858, **порт 853**) — две имплементации шифрованного DNS. В контексте обхода РФ-блокировок:
- **DoH** прячется в общем HTTPS → DPI не может тривиально срезать по порту → **partial** (работает с Yandex, частично с Cloudflare).
- **DoT** на отдельном порту 853 → **broken на мобильных РФ**, inconsistent на домашних (порт сразу видно).
- **Ни тот, ни другой** не решает SNI/IP-фильтрацию **самого приложения** после DNS-step.

## Сравнение

| Параметр | **DoH** ([[DoH в РФ]]) | **DoT** ([[DoT в РФ]]) |
|---|---|---|
| RFC | 8484 | 7858 |
| Порт | 443 (TCP/HTTPS) | 853 (TCP/TLS) |
| Видимость для DPI | как обычный HTTPS | «вот тут точно DoT» |
| РФ-2026 mobile | partial (Yandex ✓, Cloudflare ✗) | почти всегда заблокирован |
| РФ-2026 home | в большинстве работает | inconsistent |
| Поддержка в браузерах | Firefox/Chrome/Edge | нет (только ОС) |
| Поддержка ОС | Win/Linux/Android/iOS | Android 9+, systemd-resolved |
| Защищает SNI? | нет (см. [[ECH и ESNI]]) | нет |
| Защищает IP destination? | нет | нет |

**Вывод для РФ-2026:** использовать [[DoH в РФ]] на Yandex или собственный DoH-resolver на VPS. [[DoT в РФ]] оставить только для случаев, где порт 853 точно доступен.

## Какую общую проблему они решают
До любого Encrypted-DNS провайдер видит plain-DNS-запросы → может **подменять** ответы (DNS-spoofing) или **логировать** домены. Encrypted DNS убирает оба эффекта **на DNS-step**. Но дальнейший TLS-handshake к заблокированному домену провайдер всё равно увидит по SNI ([[SNI-фильтрация]]).

## Где НЕ помогает
- Не маскирует **destination-IP** прокси/VPN-сервера.
- Не защищает от **whitelist-фильтрации** ([[Белые списки]]).
- Не закрывает **SNI-leak** в TLS-handshake → нужен [[ECH и ESNI]] (но он broken в РФ-2026).

## Связи
- **Базируется на:** [[DNS — приватность и ODNS]], [[HTTPS]], [[TLS — рукопожатие]].
- **Атомарные заметки:** [[DoH в РФ]], [[DoT в РФ]].
- **Соседи:** [[ECH и ESNI]] (защита SNI), [[DNS-туннелирование]] (DoH как несущая для туннеля).
- **Теория:** [[40-glossary/DNS over HTTPS - TLS]].

## Источники
- src-05 — эмпирически: 8.8.8.8 дропается на мобильных, Yandex DNS пропускается.
- src-06 — Encrypted DNS как часть РФ-каскада обхода.
- src-01 — Encrypted DNS среди шести способов; ECH помечен сломанным.
