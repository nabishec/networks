---
title: SQL-инъекция
aliases: ["SQL injection", "SQLi"]
type: concept
layer: security
chapter: 8
difficulty: basic
prerequisites: ["[[Web — архитектура]]"]
related: ["[[XSS]]", "[[CSRF]]"]
tags: [networking, ch08]
---
# SQL-инъекция (SQLi)

## TL;DR
Класс атак, при которых **untrusted user input** попадает в SQL-запрос **без должной обработки** → атакующий может изменить **семантику** запроса. Может: украсть всю БД, удалить таблицы, обойти аутентификацию. Защита — **prepared statements / parameterized queries** (separation of code и data) + WAF + privileges minimum. Одна из топ-уязвимостей OWASP десятилетиями.

## Какую проблему решает
Понимать главный шаблон уязвимости, который **до сих пор** массово эксплуатируется в production — даже несмотря на десятилетия осведомлённости.

## Как работает

**Уязвимый код (Python):**
```python
query = f"SELECT * FROM users WHERE name='{username}' AND pass='{password}'"
cursor.execute(query)
```

Если `username = "admin' --"`:
- Финальный SQL: `SELECT * FROM users WHERE name='admin' --' AND pass='...'`
- `--` — комментарий → остаток игнорируется.
- Логин без пароля.

**Тяжёлые случаи:**
- `username = "'; DROP TABLE users; --"` — удалит таблицу.
- `username = "' UNION SELECT credit_card FROM payments --"` — извлечь чужие данные.
- **Blind SQLi**: даже если результат не виден, по timing/боолевым ответам можно «выкачивать» по байтам.

**Виды:**
1. **Classic** (in-band) — результат в HTTP-ответе.
2. **Blind** — результат не виден, выкачивается через side channels.
3. **Out-of-band** — результат шлётся на DNS/HTTP к атакующему.
4. **Second-order** — payload сохранён, выполняется позже в другом запросе.

## Защита

### 1. Prepared statements / parameterized queries (НЕ обходимы)
```python
cursor.execute("SELECT * FROM users WHERE name=%s AND pass=%s", (username, password))
```
- DB-driver разделяет **код запроса** и **данные**.
- Невозможно изменить семантику через input.

### 2. ORM (правильно использованный)
- SQLAlchemy, Django ORM, ActiveRecord — auto-parameterize.
- **Опасно**: raw SQL внутри ORM (`cursor.execute(f"...")`) — обходит защиту.

### 3. Stored procedures
- Если параметризованы — безопасны.
- Если используют dynamic SQL внутри — снова уязвимы.

### 4. Input validation (defense in depth)
- Whitelist разрешённых символов.
- Type-checking (integer-only).
- **Не самостоятельная** защита — обходится.

### 5. Privileges minimum
- DB-user веб-приложения: только SELECT/INSERT/UPDATE на конкретных таблицах.
- DROP/ALTER — отдельный admin-user.
- При SQLi atak'ер ограничен правами этого user.

### 6. WAF (Web Application Firewall)
- Регулярки для типичных payload'ов (`union`, `'--`, `or 1=1`).
- Defense in depth, не панацея.

## Пример
**Heartland Payment Systems 2008:**
- SQLi на корпоративном веб-сайте → доступ к payment-БД.
- 134 миллиона украденных credit-card numbers.
- Один из крупнейших breach'ей.

**Sony PSN 2011:**
- SQLi (одна из векторов) → 77 миллионов аккаунтов скомпрометированы.

**Defense вечная — разработчики продолжают писать уязвимый код:**
- Stack Overflow с примером `f"SELECT * FROM ... '{var}'"`.
- Junior'ы копируют без понимания.

## Связи
- **Базируется на:** [[Web — архитектура]] (где живут уязвимые apps).
- **Используется в:** OWASP Top 10 (десятилетия в топ-3).
- **Соседи по уровню:** [[XSS]], [[CSRF]] — другие web-атаки; **command injection** — родственная категория для shell.
- **Противопоставляется:** safe API design — prepared statements делают SQLi невозможной.

## Подводные камни
- **«Я escape'ю кавычки»** — ошибка. Numeric injection (`id=1 OR 1=1`) обходит quote-escape.
- **`mysql_real_escape_string`** в PHP — недостаточно при некоторых charset-настройках.
- **NoSQL injection** существует тоже — MongoDB-shell-injection через `{$gt: ""}` payloads.
- В **GraphQL** есть свои injection-классы (resolver-injection).

## Дальше читать
- [[XSS]], [[CSRF]] — другие web-атаки.
- Tanenbaum, гл. 8, §8.12.1 (стр. PDF 927–928).
