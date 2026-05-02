---
title: Глава 6. Транспортный уровень
type: moc
layer: transport
chapter: 6
tags: [networking, moc, ch06]
---
# Глава 6. Транспортный уровень — карта

Tanenbaum, гл. 6 (стр. PDF 563–683). End-to-end надёжность поверх ненадёжной сети.

## Базовое
- [[Транспортный уровень]] — задачи, отличие от L3
- [[Сокеты Беркли]] — API
- [[Порт]] — 16-бит идентификатор приложения
- [[Three-way handshake]]
- [[Разрыв соединения]]
- [[Управление потоком vs контроль перегрузки]]
- [[AIMD]]

## Протоколы
- [[UDP]] — без соединения
- [[RPC]] — модель удалённого вызова
- [[RTP]] — real-time медиа
- [[TCP]] — надёжный байтовый поток

## TCP детали
- [[TCP — заголовок]]
- [[TCP — состояния]]
- [[TCP — раздвижное окно]]
- [[Karn algorithm]]
- [[TCP — slow start]]
- [[TCP — congestion avoidance]]
- [[CUBIC TCP]]
- [[BBR]]

## Современное
- [[QUIC]]
- [[Сжатие заголовков]]
- [[Производительность LFN]]

## Дополнительно (Phase F coverage)
- [[Восстановление после сбоев]] — crash recovery, end-to-end argument.
- [[TCP fast path и header prediction]] — Van Jacobson optimization.
- [[SCTP и MPTCP]] — будущее TCP, multi-stream/multi-path.
- [[Проектирование хостов для быстрых сетей]] — DPDK, kernel bypass.

## Куда дальше
- [[ch07-application-layer]] — прикладной уровень
- [[ch05-network-layer]] — назад к L3
