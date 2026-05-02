---
title: Cookies и web tracking
aliases: ["HTTP cookies", "Set-Cookie", "Third-party cookies", "Fingerprinting"]
type: concept
layer: application
chapter: 7
difficulty: basic
prerequisites: ["[[HTTP]]"]
related: ["[[HTTPS]]", "[[Защита персональной информации]]"]
tags: [networking, ch07]
---
# Cookies и web tracking

## TL;DR
**Cookie** — кусок данных, который сервер просит браузер хранить и слать обратно при каждом запросе на тот же домен (`Set-Cookie` ↔ `Cookie`). Используется для сессий (логин), персонализации, **трекинга** (third-party). **Fingerprinting** — идентификация без cookies, по комбинации параметров браузера/системы. Главные инструменты privacy в современном вебе.

## Какую проблему решает
HTTP **stateless**: серверу нужно как-то «помнить» пользователя между запросами (логин, корзина, preferences). Cookies — самое простое решение: сервер ставит уникальный ID, браузер отдаёт его обратно. Цена — те же cookies используются для трекинга поведения.

## Как работает

**Set-Cookie** в HTTP-ответе:
```
Set-Cookie: session=abc123; Domain=example.com; Path=/; Max-Age=3600;
            Secure; HttpOnly; SameSite=Lax
```

**Атрибуты:**
- **Domain, Path** — для каких URL отдавать.
- **Max-Age / Expires** — TTL (без них — session cookie, удалится при закрытии браузера).
- **Secure** — только по HTTPS.
- **HttpOnly** — не доступен из JavaScript (защита от XSS-кражи).
- **SameSite** — `Strict / Lax / None`. Защита от CSRF + блокировка third-party.

**Cookie** в каждом следующем запросе:
```
Cookie: session=abc123
```

**Third-party cookies (3PC):**
- Страница `news.com` загружает рекламу с `ads.com`.
- `ads.com` ставит свой cookie — третья сторона.
- На любом другом сайте с рекламой `ads.com` — тот же cookie → трекинг между сайтами.
- **2024+:** Chrome заявил отказ от 3PC; Safari, Firefox блокировали раньше.

**Fingerprinting** (без cookies):
- User-Agent, screen resolution, fonts, timezone, language, plugins, canvas/WebGL hash.
- Комбинация даёт **уникальный отпечаток** для большинства пользователей.
- Pa-passive trackers собирают это вместо cookies.

## Пример
**Login flow:**
1. POST /login с username/password.
2. Server: Set-Cookie: session=ABC; HttpOnly; Secure; SameSite=Strict.
3. Каждый последующий запрос несёт cookie → server identifies user.
4. Logout: Set-Cookie с Max-Age=0 → удаление.

**Реклама/трекинг:**
- Открываешь news.com → 50 third-party скриптов трекеров.
- Каждый ставит свой ID-cookie → строит профиль "this user reads news, then went to forum".
- Подаёт целевую рекламу.

## Связи
- **Базируется на:** [[HTTP]] (механизм Set-Cookie/Cookie headers).
- **Используется в:** logins, sessions, rec engines, advertising; [[Защита персональной информации]] — связанная тема.
- **Соседи по уровню:** **localStorage / IndexedDB** в JS — клиентское хранение без отправки на сервер автоматически.
- **Противопоставляется:** stateless API с tokens в Authorization header (вместо cookies) — современный API-pattern.

## Подводные камни
- **HttpOnly** и **Secure** должны ставиться **всегда** для session cookies — иначе кража.
- **SameSite=Lax** — современный default, блокирует CSRF в большинстве случаев.
- **GDPR/cookie law:** EU требует cookie consent для не-essential cookies. Отсюда баннеры на каждом сайте.
- **Privacy Sandbox** (Google), **Topics API** — попытки заменить 3PC «privacy-respecting» альтернативами; контроверсиально.

## Дальше читать
- [[HTTP]] — механика.
- [[Защита персональной информации]] — privacy перспектива.
- Tanenbaum, гл. 7, §7.3.5 (стр. PDF 753–758).
