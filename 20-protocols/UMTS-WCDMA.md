---
title: UMTS / WCDMA
aliases: ["UMTS", "WCDMA", "3G", "HSPA"]
type: protocol
layer: physical
chapter: 2
difficulty: intermediate
prerequisites: ["[[Поколения сотовой связи 1G–5G]]", "[[Расширение спектра — DSSS]]"]
related: ["[[GSM]]", "[[LTE]]"]
tags: [networking, ch02]
---
# UMTS / WCDMA — 3G

## TL;DR
**UMTS** (Universal Mobile Telecommunications System) — европейская версия 3G от 3GPP, на основе **WCDMA** (Wideband CDMA) в радио. 5 МГц-каналы, **CDMA**-доступ (вместо TDMA в GSM). **HSPA** (3.5G, 2005-08): downlink/uplink optimization, до 42 Мбит/с. Был мостом между голос-ориентированной 2G и all-IP 4G LTE. К 2026 г. в большинстве стран отключается.

## Какую проблему решает
GSM (2G) хорош для голоса, но дает 9.6-384 кбит/с — мало для интернета. UMTS добавил **широкополосный** мобильный data (CDMA даёт лучшую спектральную эффективность чем TDMA при многих пользователях).

## Как работает

**WCDMA-радио:**
- 5 МГц-канал (vs 200 кГц в GSM).
- DSSS с chip-rate 3.84 Mcps.
- Каждый пользователь — свой scrambling-code; все на одной частоте параллельно.
- Soft handoff (пиковое преимущество CDMA): MS одновременно общается с двумя cells при handover'е.
- **Power control 1500 раз/с** — каждый MS подстраивает мощность, чтобы не интерферировать соседям.

**Архитектура (упрощённо):**
- **UE** (User Equipment) — телефон.
- **NodeB** — базовая станция WCDMA (≈ BTS в GSM).
- **RNC** (Radio Network Controller) — координирует NodeB, делает handover.
- **CN (Core Network):** circuit (MSC, для голоса) + packet (SGSN/GGSN, для data) — две параллельные плоскости.

**HSPA (High-Speed Packet Access):**
- HSDPA (downlink, 2005): до 14.4 Мбит/с.
- HSUPA (uplink, 2007): до 5.7 Мбит/с.
- HSPA+ (2008+): до 42 Мбит/с.
- 16-QAM/64-QAM, MIMO, shared channel scheduling.

**CDMA2000 / EV-DO** — параллельный 3G в США, Корее, Японии (от Qualcomm). UMTS доминировал в Европе и большей части мира.

## Пример
**Звонок и data в 3G UMTS:**
- Voice: circuit-switched путь через MSC.
- Data: packet-switched через SGSN → GGSN → интернет.
- **Параллельно:** можно говорить и качать одновременно (улучшение vs GSM).

В **смешанных GSM/UMTS-сетях** телефон попеременно использует 2G/3G в зависимости от загруженности и покрытия.

## Связи
- **Базируется на:** [[Расширение спектра — DSSS]] (WCDMA = DSSS на 5 МГц), [[Сотовая сеть — соты и handoff]].
- **Используется в:** 3G-период 2003-2018; основа для понимания эволюции к [[LTE]].
- **Соседи по уровню:** [[GSM]] (предшественник), [[LTE]] (преемник, all-IP, OFDMA вместо CDMA).
- **Противопоставляется:** GSM (TDMA) — менее эффективен в data; LTE (OFDMA) — современнее.

## Подводные камни
- **CDMA в 3G vs CDMA в WLAN** — разные техники с тем же названием. UMTS WCDMA — асинхронный, no time-sync; cdmaOne synchronous.
- **3G shutdown:** многие операторы отключают 3G к 2024-2026, оставляя 2G (для legacy IoT) и 4G/5G. **GSM-only IoT-устройства** (старые SIM-card телеметрии) могут перестать работать.
- **Soft handoff** UMTS — преимущество перед hard handoff GSM/LTE; но требовал больше state в network → сложнее core.

## Дальше читать
- [[Поколения сотовой связи 1G–5G]] — общая эволюция.
- [[GSM]] — предшественник.
- [[LTE]] — преемник.
- Tanenbaum, гл. 2, §2.6.5 (стр. PDF 200–204).
