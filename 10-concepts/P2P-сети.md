---
title: P2P-сети
aliases: ["BitTorrent", "DHT", "Kademlia", "Chord", "Peer-to-peer"]
type: concept
layer: application
chapter: 7
difficulty: intermediate
prerequisites: ["[[Клиент-сервер vs P2P]]"]
related: ["[[CDN — сеть доставки контента]]"]
tags: [networking, ch07]
---
# P2P-сети — Peer-to-Peer

## TL;DR
Equals-to-equals — каждый узел и **клиент**, и **сервер**. Контент распределён между **множеством** пиров; никто не «владеет» им центрально. Главные технологии: **BitTorrent** (файлообмен), **DHT** (Distributed Hash Table — Kademlia, Chord) для discovery без центрального трекера, **gossip-протоколы** для распространения. Применения: файлы, блокчейн, мессенджеры (Tox, Briar).

## Какую проблему решает
Клиент-серверная модель упирается в **server-side bandwidth**: миллион клиентов → миллион копий на исходящий канал сервера. P2P переворачивает: чем больше клиентов, тем больше **suммарных** ресурсов системы. Идеально для **массового распространения**, особенно legal (Linux ISO) и semi-legal (медиа).

## Как работает

### BitTorrent (Bram Cohen, 2001)
- **Файл нарезается на pieces** (~256 КБ each), у каждого SHA-1.
- **Torrent-файл** или **magnet-link** содержит список piece-hashes и tracker-URL.
- **Tracker** (или DHT) — list of peers seeding this file.
- Клиент:
  - Загружает torrent → spisok пиров.
  - Запрашивает rare pieces first (rarest-first scheduling).
  - **Tit-for-tat:** отдаёт лучше тем, кто отдаёт ему (стимул seeding'а).
- **Seeder** — пир, имеющий весь файл и продолжающий раздавать.
- **Leecher** — ещё качающий.

### DHT — distributed hash table
**Kademlia** (Maymounkov, Mazières, 2002) — самая популярная.
- Каждый узел и каждый ключ имеют **160-bit ID**.
- «Расстояние» = **XOR** ID-ов.
- Node responsible for key K — node closest to K.
- Каждый узел держит **k-buckets** ближайших соседей.
- Lookup в O(log N) через iterative XOR-routing.

**Chord** (Stoica et al., 2001) — академический предшественник, кольцевая структура.

### Gossip-протоколы
- Каждый узел случайно выбирает другого и обменивается state.
- Сходится к consistent globally в O(log N) раундов.
- Основа Cassandra, Consul, blockchain.

## Пример
- **Linux ISO release:** Ubuntu публикует .iso через torrent. Миллионы скачивают; нагрузка на сервер мала, потому что peers сами отдают друг другу.
- **BitTorrent magnet link:** `magnet:?xt=urn:btih:HASH&dn=name`. Клиент через DHT находит peers без tracker'а.
- **Bitcoin:** P2P-сеть из десятков тысяч узлов; каждый держит копию blockchain; gossip-распространение блоков.
- **WebTorrent:** BitTorrent в браузере через WebRTC.

## Связи
- **Базируется на:** [[Клиент-сервер vs P2P]] (антагонистическая модель).
- **Используется в:** torrents, blockchain, decentralized social networks (Mastodon — federated, не строго P2P), IPFS.
- **Соседи по уровню:** [[CDN — сеть доставки контента]] — централизованная альтернатива; **federated systems** — middle ground (Mastodon, Matrix).
- **Противопоставляется:** клиент-сервер — централизованная модель.

## Подводные камни
- **NAT traversal:** многие peers за NAT — нужны TURN-relay или hole punching через STUN.
- **Free riders:** пользователи, скачивающие но не отдающие. BitTorrent смягчает через tit-for-tat.
- **Legal/copyright:** P2P часто ассоциируется с пиратством — legitimate use cases (Linux distros) затмеваются.
- **DHT spam/sybil-атаки:** атакующий может создать миллионы фальшивых node-IDs → влиять на routing. Решения сложные.

## Дальше читать
- [[Клиент-сервер vs P2P]] — фундаментальное противопоставление.
- [[CDN — сеть доставки контента]] — централизованная alternativa.
- Tanenbaum, гл. 7, §7.5.4 (стр. PDF 797–803).
