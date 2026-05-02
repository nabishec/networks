---
title: Глава 4. MAC-подуровень
type: moc
layer: mac
chapter: 4
tags: [networking, moc, ch04]
---
# Глава 4. MAC-подуровень — карта

Tanenbaum, гл. 4 (стр. PDF 312–411). На разделяемых средах нужен арбитраж: «**кто говорит сейчас**». Это и есть Medium Access Control.

## Базовое

- [[Подуровень MAC]] — задачи: арбитраж разделяемой среды.
- [[Проблема распределения канала]] — статика vs динамика.

## Random access протоколы

- [[ALOHA]] — pure / slotted; первый протокол случайного доступа.
- [[CSMA]] — с прослушиванием несущей: 1-persistent / non-persistent / p-persistent.
- [[CSMA/CD]] — с обнаружением коллизий; основа classic Ethernet.
- [[CSMA-CA|CSMA/CA]] — с избеганием коллизий; основа Wi-Fi.

## Безконфликтные

- [[Bit-map и token-passing]] — Token Ring, FDDI, упорядоченный доступ.

## Беспроводные проблемы

- [[Hidden terminal problem]]
- [[Exposed terminal problem]]
- [[RTS/CTS]] — решение hidden terminal.
- [[NAV — Network Allocation Vector]] — virtual carrier sensing.

## Ethernet (802.3)

- [[Ethernet — IEEE 802.3]] — формат фрейма, MAC-адрес.
- [[MAC-адрес]] — 48-бит, OUI+NIC.
- [[Коммутируемый Ethernet]] — switch вместо хаба.
- [[Эволюция Ethernet]] — 10/100/1G/10G/40G/100G.

## Wi-Fi (802.11)

- [[802.11 — Wi-Fi архитектура]] — STA, AP, BSS, ESS.
- [[802.11 MAC — DCF]] — Distributed Coordination Function.
- См. также [[Wi-Fi — обзор]] (гл. 1).

## Bluetooth

- [[Bluetooth]] — piconet/scatternet, master/slave, BLE.

## Дополнительно (medium-coverage)

- [[Adaptive tree walk]] — limited-contention, log₂(N) разрешение коллизий.
- [[Binary countdown]] — bit-arbitrage, основа CAN-bus.
- [[DOCSIS MAC]] — ranging, мини-слоты, request-grant.
- [[Службы 802.11]] — auth, association, distribution.

## Коммутаторы и сегментация L2

- [[Мост и обучающийся мост]] — bridge, MAC learning.
- [[Spanning Tree Protocol]] — устранение петель.
- [[Hub vs Switch vs Router vs Gateway]] — кто на каком уровне.
- [[VLAN — IEEE 802.1Q]] — виртуальные LAN.

## Куда дальше

- [[ch05-network-layer]] — сетевой уровень.
- [[ch03-data-link-layer]] — назад к LLC.
