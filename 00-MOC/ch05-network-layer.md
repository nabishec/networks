---
title: Глава 5. Сетевой уровень
type: moc
layer: network
chapter: 5
tags: [networking, moc, ch05]
---
# Глава 5. Сетевой уровень — карта

Tanenbaum, гл. 5 (стр. PDF 412–562). End-to-end маршрутизация пакетов через много сетей. Самый большой раздел книги.

## Базовое
- [[Сетевой уровень]] — задачи: маршрутизация end-to-end.
- [[Datagram subnet vs virtual-circuit subnet]] — IP vs ATM/MPLS.

## Алгоритмы маршрутизации
- [[Принцип оптимальности маршрутизации]]
- [[Алгоритм Дейкстры]] — shortest path
- [[Лавинная маршрутизация (flooding)]]
- [[Distance Vector Routing]]
- [[Count-to-infinity problem]]
- [[Link State Routing]]
- [[Иерархическая маршрутизация]]
- [[Multicast routing]]
- [[Anycast]]

## Управление трафиком
- [[Перегрузка сети]]
- [[Token bucket]]
- [[Leaky bucket]]
- [[RED и AQM]]
- [[ECN]]

## QoS
- [[QoS vs QoE]]
- [[IntServ и RSVP]]
- [[DiffServ]]

## Internetworking
- [[Internetworking — фрагментация]]
- [[Туннелирование]]

## SDN
- [[SDN — программно-конфигурируемые сети]]
- [[OpenFlow]]

## Сетевой уровень интернета
- [[IPv4]] — заголовок, опции
- [[IP-адресация и CIDR]] — CIDR, longest-prefix
- [[NAT]] — Network Address Translation
- [[IPv6]] — 128 бит
- [[ICMP]] — служебные сообщения
- [[ARP]] — IP→MAC
- [[DHCP]] — автоконфигурация
- [[MPLS]] — label switching
- [[OSPF]] — IGP link-state
- [[BGP]] — EGP path-vector
- [[Multicast Internet]] — IGMP, PIM

## Дополнительно (medium-coverage)

- [[Multidestination routing]] — list получателей в пакете.
- [[Congestion collapse и goodput]] — throughput vs goodput, история 1986.
- [[Автономная система]] — ASN, types, иерархия.
- [[Пиринговый спор]] — Netflix vs Comcast, de-peering.
- [[Proxy ARP]] — proxy answers, mobile IP, RARP/BOOTP.

## Куда дальше
- [[ch06-transport-layer]] — транспортный уровень.
- [[ch04-mac-sublayer]] — назад к MAC.
