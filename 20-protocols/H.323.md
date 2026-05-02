---
title: H.323
aliases: ["H.323", "Gatekeeper"]
type: protocol
layer: application
chapter: 7
difficulty: intermediate
prerequisites: ["[[VoIP и SIP]]"]
related: ["[[RTP]]"]
tags: [networking, ch07]
---
# H.323 — стек интернет-телефонии (ITU-T)

## TL;DR
**ITU-T-стандарт** для VoIP и видеоконференций (1996, до SIP). Многослойный, монолитный — определяет всё: gatekeeper-функции, шлюзы, сигнализацию (Q.931), capability exchange (H.245), media (RTP/RTCP). 1400 страниц спецификации vs 250 у SIP. **Проиграл SIP** на интернете из-за сложности; до сих пор в legacy-операторских системах и видеоконференциях (Polycom).

## Какую проблему решает
Когда H.323 разрабатывался (1996), VoIP был новинкой. ITU подошёл с привычной телекоммуникационной парадигмой: **полный стек**, аналог ISDN-Q.931 поверх IP. Цель — мост IP-сетей с PSTN, поддерживающий все возможные сценарии.

Тяжелее, но **полнее** на момент создания.

## Как работает

**Архитектура:**
- **Терминал** — IP-телефон или софт-клиент.
- **Шлюз (Gateway)** — между H.323 и PSTN/SCN.
- **Привратник (Gatekeeper)** — координация: address translation (E.164 → IP), admission control, bandwidth management. Опционален, но в больших развёртываниях стандарт.
- **MCU** (Multi-point Control Unit) — для конференций.
- **Зона** — все терминалы под одним gatekeeper.

**Стек протоколов:**
```
+--------+---------+--------+---------+
|G.711/  | H.261/  | RAS    | H.225 / |
|G.729   | H.263 / | (UDP)  | Q.931   |
|        | H.264   |        | (TCP)   |
+--------+---------+--------+---------+
|         RTP/RTCP (UDP)             |
+------------------------------------+
|              UDP / TCP              |
+------------------------------------+
|                IP                   |
+------------------------------------+
```

- **H.225 / Q.931** — call signaling (SETUP, CALL PROCEEDING, ALERTING, CONNECT, RELEASE COMPLETE).
- **H.245** — capability exchange (which codec? master/slave?).
- **RAS** (Registration, Admission, Status) — между endpoint и gatekeeper.
- **G.7xx, H.26x** — кодеки голоса/видео.
- **RTP/RTCP** — собственно медиа.

**Flow установления вызова:**
1. RAS-Регистрация у gatekeeper.
2. ARQ (Admission Request) — gatekeeper проверяет квоту.
3. Q.931 SETUP к destination.
4. CALL PROCEEDING / ALERTING / CONNECT.
5. H.245 capability negotiation.
6. RTP-media flow.

## Пример
**Корпоративная видеоконференция Polycom (2010s):**
- Кабинет с Polycom HDX — H.323-терминал.
- Регистрируется в gatekeeper (Cisco CallManager или сторонний).
- Другой кабинет в другом городе — тот же gatekeeper или federated.
- Звонок: SETUP → ALERTING → CONNECT → H.245 → RTP HD video.
- Qos через DiffServ EF.

**Постепенный закат H.323:**
- WebRTC и cloud-conferencing (Zoom, Teams) используют свои протоколы.
- SIP замещает H.323 в новых deployments.
- H.323 ещё в operator backbones (SS7-IP миграция) и enterprise legacy.

## Сравнение с SIP

| | H.323 | SIP |
|---|---|---|
| Авторство | ITU-T | IETF |
| Размер спецификации | ~1400 страниц | ~250 страниц |
| Стиль | монолитный, telco-style | модульный, web-style |
| Текстовый/бинарный | бинарный (ASN.1) | текстовый (HTTP-like) |
| Расширяемость | сложно | легко (headers) |
| Адресация | E.164 + alias | URI (sip:user@host) |
| Победил на | enterprise legacy | весь интернет |

## Связи
- **Базируется на:** [[RTP]] (media), [[Транспортный уровень]] (TCP/UDP).
- **Используется в:** Cisco CallManager (legacy), Polycom, ISDN-IP gateways операторов.
- **Соседи по уровню:** [[VoIP и SIP]] — главный соперник.
- **Противопоставляется:** SIP — выиграл на интернете.

## Подводные камни
- **Bilingual deployments:** многие enterprise-системы поддерживают **оба** (gateway между H.323 и SIP).
- **NAT-traversal в H.323** — болезненно (использует много портов: Q.931, H.245, RTP, RTCP — все разные).
- **Gatekeeper SPOF:** падение gatekeeper'а — звонки не идут.

## Дальше читать
- [[VoIP и SIP]] — победитель.
- [[RTP]] — media transport.
- Tanenbaum, гл. 7, §7.4.4 (стр. PDF 776–784).
