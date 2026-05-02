---
title: Именование доменов — ICANN
aliases: ["ICANN", "Cybersquatting", "TLD politics", "gTLD"]
type: concept
layer: application
chapter: 7
difficulty: basic
prerequisites: ["[[DNS]]"]
related: ["[[Стандартизация сетей]]"]
tags: [networking, ch07]
---
# Именование доменов — ICANN, TLD-политика, киберсквоттинг

## TL;DR
**ICANN** (Internet Corporation for Assigned Names and Numbers, 1998) — некоммерческая организация, управляющая корневой DNS-зоной и распределением TLD. Решает: какие новые **gTLD** (generic — `.com`, `.xyz`, `.app`) создавать, как распределять **ccTLD** (country-code — `.ru`, `.tv`). **Cybersquatting** — регистрация чужих торговых марок как доменов для перепродажи; решается через **UDRP** (Uniform Domain-Name Dispute-Resolution Policy). Глубоко политизированная сфера.

## Какую проблему решает
Доменные имена — экономический и политический ресурс. Без центральной координации:
- Хаос: два «владельца» одного `.com`-имени.
- Cybersquatters блокируют trademarks.
- Геополитика: «кто контролирует `.ru`?»

ICANN — попытка multi-stakeholder governance, не идеальная, но работает.

## Как работает

**Иерархия:**
- **IANA** (Internet Assigned Numbers Authority) — базовая функция (root zone, ASN, IP allocation), теперь под ICANN.
- **ICANN board** — multi-stakeholder (governments, companies, civil society).
- **Registries** управляют TLD: VeriSign (.com), Public Interest Registry (.org), .ru координируется КЦ Минцифры.
- **Registrars** продают домены конечным пользователям (GoDaddy, REG.RU, Namecheap).

**Типы TLD:**

| Тип | Пример | Управление |
|---|---|---|
| **gTLD** generic | .com, .org, .net | mostly US-based registries |
| **ccTLD** country | .ru, .uk, .jp | национальная организация |
| **sTLD** sponsored | .gov, .edu, .museum | специальные критерии |
| **new gTLD** (с 2014) | .app, .xyz, .pizza | расширение namespace |

**Финансы TLD:**
- **.tv** Tuvalu сдаёт в аренду (Verisign контракт ~$5M/год + перепродажа стримерам).
- **.io** — British Indian Ocean Territory; Internet companies любят за красивое имя.
- **.me** — Montenegro.
- Маленькие страны зарабатывают миллионы на «модных» суффиксах.

**Cybersquatting:**
- 1990s: компании опаздывали зарегистрировать свои бренды → squatters делали это первыми и продавали по высоким ценам.
- **UDRP** (с 1999): арбитраж по спорам о доменах. Trademark holder может вернуть домен, если докажет bad faith.
- Современно: компании регистрируют десятки variations превентивно.

**ICANN-кризисы:**
- 2003: создание `.xxx` — глобальный спор о censorship.
- 2014: massive new gTLD round (`.guru`, `.attorney`, …) — много непродаваемых TLD.
- 2016: USA передала контроль над IANA полностью под ICANN (international); до этого был USDOC.

## Пример
- **Apple Inc. vs apple.com:** apple.com был зарегистрирован Apple Computer быстро, no dispute.
- **Madonna.com case (2000):** UDRP, певица победила сквоттера.
- **Russian transliterations:** `апл.рф` — кирилличный TLD `.рф` запущен 2010, медленно adoption'ился.

## Связи
- **Базируется на:** [[DNS]] (что именуется), [[Стандартизация сетей]] (governance).
- **Используется в:** все web-сервисы — у каждого есть домен от какого-то TLD.
- **Соседи по уровню:** **WHOIS** (RDAP сейчас) — реестр владельцев доменов.
- **Противопоставляется:** «free naming» — нереалистично; **alternative roots** (типа OpenNIC) маргинальны.

## Подводные камни
- **Trademark vs free speech** часто конфликтует: критический сайт `microsoft-sucks.com` — Microsoft пытался получить через UDRP, но обычно проигрывает.
- **Domain seizure** — государство может отобрать домен через registry. ICE seizures dot-com. Russian «Russian internet» discussions.
- **Punycode** для IDN (Internationalized Domain Names) — `xn--80akhbyknj4f.рф`, потенциальные phishing-атаки через Unicode-confusables.

## Дальше читать
- [[DNS]] — техническая основа.
- [[Стандартизация сетей]] — governance contexts.
- Tanenbaum, гл. 7, §7.1.8 (стр. PDF 704).
