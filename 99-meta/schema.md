---
title: Schema
type: meta
tags: [meta, schema]
---
# Schema базы знаний

Источник: Таненбаум Э., Фимстер Н., Уэзеролл Д. «Компьютерные сети», 6-е изд. (Питер, 2023, ISBN 978-5-4461-1766-6).

## Папки

- `00-MOC/` — точки входа (Map of Content): общий index, by-layer, by-chapter, learning-paths.
- `10-concepts/` — атомарные концепты-идеи (одна идея — один файл): «принцип оптимальности», «коммутация пакетов», «hidden terminal problem» и т.п.
- `20-protocols/` — протоколы: TCP, IP, HTTP, BGP, OSPF, ARP, DHCP, DNS, IPsec, TLS, …
- `30-algorithms/` — алгоритмы: Dijkstra, Distance Vector, Go-Back-N, ALOHA, CSMA/CD, AIMD, Karn, …
- `40-glossary/` — короткие определения и заглушки для нерешённых [[wikilinks]].
- `99-meta/` — schema, template, build-log, plan, chapters.json.
- `libraries/` — исходные файлы книги (PDF/EPUB/DOCX). Не индексировать смыслово.

## Типы заметок (frontmatter `type`)

- `concept` — общая идея, принцип, проблема (10-concepts).
- `protocol` — конкретный сетевой протокол (20-protocols).
- `algorithm` — конкретный алгоритм (30-algorithms).
- `layer` — заметка-обзор уровня модели (00-MOC).
- `device` — устройство (коммутатор, маршрутизатор) (10-concepts).
- `term` — короткое определение, заглушка (40-glossary).
- `moc` — карта контента (00-MOC).
- `meta` — схема, шаблон, лог (99-meta).

## Уровни (frontmatter `layer`)

`physical | data-link | mac | network | transport | application | security | cross-layer`

`security` — отдельная категория (глава 8 Tanenbaum'а), пересекает все уровни.
`mac` — подуровень канального уровня (глава 4), отделён, потому что у Tanenbaum'а отдельная глава.
`cross-layer` — для понятий, не привязанных к одному уровню (адресация, RPC, QoS).

## Языковые правила

- Русский. Англ. термин в скобках при первом упоминании: «коммутатор (switch)».
- Аббревиатуры расшифровываются один раз: «BGP (Border Gateway Protocol)».
- Цитаты из книги — короткие, в кавычках, с указанием стр. PDF: «…цитата…» (стр. 487).
- Никакого копирования абзацами. Своими словами, опираясь на источник.

## Источники

- **PDF (основной для accuracy):** `libraries/Tanenbaum_E__..._-_2023.pdf`, 992 стр., текстовый слой OK.
- **EPUB (основной для парсинга):** `libraries/Tanenbaum_E__..._-_2023.epub`, чистый XHTML.
- **DOCX (доп. cross-check):** `libraries/Tanenbaum_E__..._-_2023.docx` — пользователь добавил, чтобы ничего не упустить.

При расхождениях между EPUB и PDF — **PDF приоритетнее** (исходный набор).
