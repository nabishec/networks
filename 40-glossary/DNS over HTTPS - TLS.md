---
title: DNS over HTTPS / TLS
aliases: ["DNS over HTTPS / TLS", "DoH", "DoT"]
type: term
layer: application
chapter: 7
difficulty: basic
prerequisites: []
related: []
tags: [networking, ch07, stub]
---
# DNS over HTTPS / TLS

> ⚠ Заглушка — детально в гл. 8 (privacy).

DoH (RFC 8484) — DNS-запросы по HTTPS на порт 443 (виглядит как обычный веб-трафик). DoT (RFC 7858) — на отдельном порту 853 поверх TLS. Цель — защита от подслушивания и spoofing'а DNS со стороны провайдера.

## Дальше читать
- Tanenbaum, гл. 7, §7.1.7.
