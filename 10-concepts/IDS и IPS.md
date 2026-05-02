---
title: IDS и IPS
aliases: ["Intrusion Detection System", "Intrusion Prevention System", "Snort", "Suricata"]
type: concept
layer: security
chapter: 8
difficulty: basic
prerequisites: ["[[Брандмауэр]]"]
related: ["[[Виды атак]]"]
tags: [networking, ch08]
---
# IDS и IPS — Intrusion Detection / Prevention Systems

## TL;DR
**IDS** — пассивно мониторит трафик, **сигнализирует** о подозрительной активности (alerts). **IPS** — активно **блокирует** обнаруженное (in-line). Два метода детекции: **signature-based** (правила для известных атак) и **anomaly-based** (отклонения от baseline'а — ML/statistics). Tools: Snort, Suricata, Zeek, commercial (Palo Alto, Cisco).

## Какую проблему решает
[[Брандмауэр]] — basic фильтр по 5-tuple. Не различает «легитимный HTTP» и «HTTP-инъекция SQL»; не видит exfiltration через DNS-tunneling. IDS/IPS добавляет **углубленную инспекцию** — детектирует характерные паттерны атак, аномалии.

## Как работает

### Signature-based detection
- БД правил (rules): «если пакет содержит pattern X — alert».
- Snort-правило example:
```
alert tcp any any -> $HOME_NET 80 (msg:"SQL injection attempt"; 
content:"' or 1=1"; nocase; sid:1000001;)
```
- Плюс: low false-positive на known attacks.
- Минус: **не ловит** zero-day и mutated attacks.

### Anomaly-based detection
- Учится «что нормально» (baseline) → отклонения = alert.
- ML/statistics: NetFlow analysis, peer-traffic comparison.
- Плюс: ловит **unknown** attacks.
- Минус: высокий false-positive rate, особенно при переменчивом трафике.

### Deployment

| Тип | Где стоит | Что видит |
|---|---|---|
| **NIDS/NIPS** (network) | inline или mirror port | сетевой трафик |
| **HIDS/HIPS** (host) | на endpoint | system calls, file changes, logs |
| **Hybrid** | сочетание | оба |

### IDS vs IPS

| | IDS (passive) | IPS (active) |
|---|---|---|
| Размещение | mirror / SPAN-port | inline (in path) |
| Действие | alert | block + alert |
| Latency | 0 | small (inspection time) |
| False-positive cost | звонок ночью | заблокированный legitimate user |

## Пример
**Suricata в офисе:**
- Inline на edge-router.
- Снифит весь HTTP/DNS-трафик.
- Signature-rules от Emerging Threats (free).
- Alert на: SQL-injection patterns, malware-C2 communications, suspicious DNS-queries.
- При IPS-mode: блокирует.

**Modern SOC** (Security Operations Center):
- IDS shлёт alerts в **SIEM** (Splunk, Elastic).
- Аналитики triage.
- Автоматика (SOAR) реагирует на known attack signatures.

## Связи
- **Базируется на:** [[Брандмауэр]] (часто работают вместе), DPI (deep packet inspection).
- **Используется в:** SOC, compliance (PCI-DSS требует IDS).
- **Соседи по уровню:** **WAF** (Web Application Firewall) — IPS specific для HTTP. **EDR** — host-уровень, advanced.
- **Противопоставляется:** только firewall — слишком грубо.

## Подводные камни
- **Encrypted traffic** — IDS не видит payload. Решения: TLS-mid-MITM (privacy concerns), traffic-side analysis (TLS metadata).
- **Alert fatigue:** signature-based часто шумит. Нужен SOC + tuning.
- **Bypass через encoded payload** — атакующие используют base64, fragmentation для обхода simple signatures. Modern IDS reassemble и decode.
- **IPS-block of legitimate traffic** — болезненно, особенно для critical apps. Часто IPS deploy'ят в IDS-mode сначала, потом постепенно switch.

## См. также (прикладное)
RF-circumvention: ТСПУ — государственный IDS/IPS-stack с signature-based и anomaly-based методами.
- [[ТСПУ]] — DPI-stack ФЗ-90.
- [[DPI-фильтрация в РФ]] — методы детекции (SNI, JA3/JA4, session-freezing).
- [[Active probing]] — IPS-style probing: DPI само подключается к подозрительному IP.
- [[uTLS]] — обход JA3/JA4-fingerprint-классификации.
- [[applied-rf-status]] — обзор техник обхода.

## Дальше читать
- [[Брандмауэр]] — основа.
- [[Виды атак]] — что детектируем.
- Tanenbaum, гл. 8, §8.3.2 (стр. PDF 847–852).
