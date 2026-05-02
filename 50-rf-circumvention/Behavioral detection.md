---
title: Behavioral detection (post-handshake)
aliases: ["Behavioral detection", "Post-handshake detection", "ML-классификатор VPN"]
type: concept
layer: security
status: working
status_as_of: 2026-05-02
risk: low
prerequisites: ["[[DPI-фильтрация в РФ]]", "[[ТСПУ]]", "[[TLS — рукопожатие]]"]
related: ["[[Active probing]]", "[[uTLS]]", "[[JA3-JA4 fingerprinting]]", "[[Session freezing]]"]
sources: [src-10, src-02]
tags: [networking, rf-circumvention, concept]
---
# Behavioral detection (post-handshake)

## TL;DR
DPI-метод детекции VPN/прокси, который срабатывает **после** TLS-handshake'а — когда signature-проверка не дала однозначного ответа. Система собирает **статистику** по размерам пакетов, межпакетным интервалам, energy-распределению и направлению трафика, затем классифицирует его через **ML-модель** (random-forest, gradient-boost, реже — neural net). По заявлениям src-10 и публичным отчётам, точность ML-классификатора в лабораторных условиях достигает **95–99%** для Shadowsocks, VMess, Hysteria-без-Salamander.

## Какую проблему решает (для DPI)
Современные censorship-resistant протоколы маскируются под HTTPS: ClientHello подменяется через [[uTLS]], сертификат — настоящий ([[VLESS-Reality]]), JA3 совпадает с Chrome. Signature-классификаторы дают «это HTTPS» → нечего блокировать. Но **поведение** трафика отличается от обычного веб-сёрфинга:
- VPN держит **долгоживущую** сессию (минуты-часы), браузер — короткую (секунды).
- В VPN **симметричный** объём (upload ≈ download), в вебе **асимметричный** (download >> upload).
- VPN-туннель — равномерный поток; HTTP-сёрфинг — burst'ы пауз.
- Энтропия пакетов — высокая (зашифрованный туннель внутри TLS), у обычного HTTP/2 — структурированная (HPACK).

ML-модель на этих признаках детектирует VPN с задержкой 30 секунд–5 минут после установления сессии.

## Как работает

### Признаки (features)
| Признак | Обычный браузер | VPN-туннель |
|---|---|---|
| Длительность сессии | секунды | минуты-часы |
| Симметрия upload/download | ~5/95 | ~50/50 |
| Распределение размеров пакетов | bimodal (small ACK + large MTU) | uniform |
| Inter-arrival time | burst'ы + idle | равномерный |
| Энтропия payload | средняя (mix HTML + JSON + image) | максимальная |
| Доля 1448-байтных пакетов | низкая | высокая (TUN + VPN-overhead) |
| TLS NewSessionTicket per minute | low | high (Reality keep-alive) |

### Pipeline ТСПУ (по src-10)
1. **Capture** — sFlow-агенты на узлах, экспорт meta в data-plane.
2. **Aggregation** — sliding window 30-300 секунд per (src-IP, dst-IP, dst-port).
3. **Feature extraction** — 30+ признаков на сессию.
4. **Inference** — ML-модель (offline-trained), результат: `{0..1, threshold 0.85}`.
5. **Action** — drop, throttle или enroll в greylist для углублённого анализа.

### Где даёт сбой
- **Короткие сессии** (< 30 сек) — недостаточно данных для ML.
- **Маскировка под streaming** (Hysteria-Salamander, multiplex с HTTP/3 на YouTube) — сбивает признак «равномерности».
- **Browser-in-tunnel** (VPN запускает только браузер, не другие apps) — статистика ближе к нормальной.

## Защита (анти-detection)
| Техника | Как помогает |
|---|---|
| **Salamander-обфускация** ([[Hysteria-2]]) | Random padding + jitter → ломает inter-arrival-pattern. |
| **xHTTP packet-up** ([[xHTTP]]) | Каждый upload — отдельный POST → выглядит как формы / API-вызовы. |
| **Self-Steal** ([[Self-Steal — свой домен]]) | Большая часть трафика — реальные HTTP к собственному сайту, VPN — на одном path. |
| **Mux fan-out** | Одна VPN-сессия раздаётся через 4-8 stream'ов с разными timing'ами. |
| **Краткосрочные сессии** | Auto-rotate соединений каждые 60-120 секунд — ML не успевает накопить статистику. |

## Связи
- **Базируется на:** [[DPI-фильтрация в РФ]] (общий контекст), ML/statistics — внешняя дисциплина.
- **Дополняет:** [[Active probing]] (probing — что случилось при первом подключении; behavioral — что происходит во времени).
- **Обходится через:** [[uTLS]] (помогает на handshake, но не покрывает behavioral), Salamander, [[xHTTP]], [[Self-Steal — свой домен]].
- **Маркер срабатывания:** долгая сессия → внезапный drop без RST после ~3-5 минут (см. [[Session freezing]] для 16-КБ-варианта).

## Источники / Дальше читать
- [src-10 — Не соглашаясь с Большим Братом. Telegram и MTProxy](https://habr.com/ru/articles/1018740/) — упоминание ML-классификатора и post-handshake-detection.
- [src-02 — Гайд по обходу белых списков](https://habr.com/ru/articles/985674/) — упоминание сессионного freezing'а как одного из behavioral-сигналов.
- Habr: [Как работают ТСПУ и DPI](https://habr.com/ru/articles/992232/) — ML-anomaly detection, TLS-fingerprint-БД.
- Habr: [Анатомия DPI анализа: что происходит с пакетом за первые 16 КБ](https://habr.com/ru/articles/1009560/) — ML-классификаторы 95-99% точности для Shadowsocks/VMess.
- Habr: [Как оператор связи видит, что вы используете VPN](https://habr.com/ru/articles/1017158/) — техника детекции с точки зрения провайдера.
- Habr: [Как я защитил свой VPN от DPI: graylist + nginx stream](https://habr.com/ru/articles/1022494/) — practical defense-стратегии.
- Habr: [DPI-First: почему анализ трафика становится сердцем сети](https://habr.com/ru/articles/965070/) — обзор DPI-индустрии.
