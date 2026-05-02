---
title: CSRF
aliases: ["CSRF", "Cross-Site Request Forgery"]
type: concept
layer: security
chapter: 8
difficulty: basic
prerequisites: ["[[Cookies и web tracking]]", "[[HTTP]]"]
related: ["[[XSS]]", "[[SQL-инъекция]]"]
tags: [networking, ch08]
---
# CSRF — Cross-Site Request Forgery

## TL;DR
Атака, **заставляющая залогиненного пользователя** **неосознанно** выполнить действие на сайте. Атакующий встраивает на свой сайт `<form action="https://bank.com/transfer" method="POST">...</form>` или `<img src="...">` с автодействием. Браузер жертвы автоматически шлёт **cookies** банка с запросом → банк думает, это легитимный пользователь. Защита: **CSRF-токен**, **SameSite cookies**, **Origin/Referer check**.

## Какую проблему решает
В отличие от XSS (выполнение JS на жертвенном сайте), CSRF использует **доверчивость** браузера: cookies сайта **автоматически прикрепляются** к каждому запросу к этому сайту, даже инициированному с другого домена. Если сайт не проверяет «откуда» — любой может «нажать кнопки» от имени залогиненного.

## Как работает

**Сценарий:**
1. Жертва залогинилась в `bank.com` (cookie `session=abc`).
2. Зашла на `evil.com`.
3. На evil.com:
   ```html
   <img src="https://bank.com/transfer?to=hacker&amount=10000" style="display:none">
   ```
4. Браузер автоматически запрашивает image — **с cookie от bank.com**.
5. bank.com: «о, request от authenticated user, перевожу 10000».
6. **Без участия жертвы.**

**С POST-формой:**
```html
<form action="https://bank.com/transfer" method="POST">
  <input name="to" value="hacker"><input name="amount" value="10000">
</form>
<script>document.forms[0].submit();</script>
```

## Защита

### 1. CSRF-токен (классический)
- Сервер генерирует случайный токен, кладёт в hidden field формы и в session.
- При POST проверяет, что токен совпадает.
- Атакующий **не может** прочитать токен с bank.com (Same-Origin Policy).

### 2. SameSite cookies
- `Set-Cookie: session=abc; SameSite=Lax` → cookie не отправляется при cross-site запросах.
- **Lax** (default в современных браузерах): отправляется только при top-level navigation (clicked link), не при image/iframe POST.
- **Strict**: вообще не отправляется ни при каком cross-site request.
- **None**: старое поведение (отправляется всегда), требует `Secure`.

### 3. Origin / Referer check
- Сервер: «принимать POST только если Origin/Referer совпадает с моим доменом».
- Простая защита; иногда ломается прокси / privacy-extensions.

### 4. Double-submit cookie
- Token в cookie + token в form data; сервер проверяет, что они совпадают.
- Atтакующий не может ставить cookie на чужой домен.

## Пример
**Нашумевший Twitter worm 2010 (CSRF + worm):**
- Атакующий создаёт страницу с CSRF-payload «retweet текст ...».
- При просмотре этой страницы жертва автоматически ретвитит.
- Ретвит видят followers → они смотрят страницу → ретвитят.
- Self-propagating CSRF.

**ATM PIN изменение (гипотетический):**
- evil.com hosts: `<img src="https://bank.com/change-pin?new=0000">`.
- Жертва, залогиненная в bank.com, открывает evil.com → PIN изменён на 0000.

Современный bank.com защитил через SameSite + CSRF-токен → не работает.

## Связи
- **Базируется на:** [[Cookies и web tracking]] (используется доверчивость cookies), [[HTTP]] (request mechanics).
- **Используется в:** Web security best practices.
- **Соседи по уровню:** [[XSS]] — другая web-атака; **Same-Origin Policy** — фундамент защиты.
- **Противопоставляется:** XSS — XSS позволяет выполнять JS, CSRF лишь делать запросы.

## Подводные камни
- **GET для state-changing actions** — anti-pattern (можно через `<img>`). Используйте POST/PUT/DELETE.
- **CORS не защищает от CSRF** — CORS про чтение ответа, CSRF — про сам запрос (которого было достаточно).
- **SameSite=Lax** не защищает от GET-CSRF (top-level navigation), нужны и токены.
- **API-only сайты** (без cookies, на Bearer-tokens в Authorization header) — менее уязвимы к CSRF, потому что header не auto-attaches.

## Дальше читать
- [[XSS]] — соседняя атака.
- [[Cookies и web tracking]] — что эксплуатируется.
- Tanenbaum, гл. 8, §8.12.1 (стр. PDF 927–928).
