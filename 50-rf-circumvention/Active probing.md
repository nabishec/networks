---
title: Active probing
aliases: ["Active probing", "Активное зондирование"]
type: concept
layer: application
status: working
status_as_of: 2026-05-02
risk: low
prerequisites: ["[[ТСПУ]]"]
related: ["[[VLESS-Reality]]", "[[uTLS]]"]
sources: [src-08, src-10]
tags: [networking, rf-circumvention, concept]
---
# Active probing — активное зондирование

## TL;DR
Атакующий (DPI/ТСПУ) **сам инициирует соединение** к подозрительному IP, чтобы проверить, **что там работает**. Если сервер отвечает «как VPN» (Shadowsocks-pattern, OpenVPN-handshake, etc.) → blacklist. Если отвечает «как обычный сайт» (HTTP 200 с реальным контентом) → пропуск. Защита: **Reality** (handshake к настоящему target), **Self-Steal** (свой реальный сайт), uTLS (mimicry).

## Какую проблему решает (с точки зрения регулятора)
- Найти **VPN-серверы** среди миллиардов IP — задача нерешимая через passive observation.
- Active probing: после **подозрительного flow** (VPN-pattern в трафике) DPI шлёт несколько проб → если IP отвечает «как VPN» → blacklist.

## Как работает

**Типичный Great Firewall of China-style probe** (применимо концептуально для ТСПУ):
1. Пользователь подключается к suspicious IP.
2. DPI логирует IP.
3. Через 5-30 минут (delayed) DPI с «своих» IP отправляет:
   - Random bytes к suspicious port.
   - Replay handshake'а от пользователя.
   - Specific protocol probes (SS, OpenVPN, V2Ray).
4. Если сервер отвечает «как VPN»:
   - Shadowsocks-server: ответит с decryption-error timing.
   - OpenVPN-server: ответит с TLS-handshake.
   - VLESS+TLS со своим cert: cert валидный, но SAN/issuer известны.
5. → IP в blacklist.

**Защита через Reality:**
- DPI-probe → handshake к Reality-server.
- Server: проверяет HMAC-secret. Не совпадает → forward к **настоящему target** (microsoft.com).
- DPI получает ответ настоящего microsoft.com → «обычный TLS-сайт» → пропуск.

**Защита через Self-Steal:**
- DPI-probe → HTTP-запрос на /.
- Server: nginx отдаёт реальный landing-page.
- DPI: «обычный сайт».

## Где ломается / почему может не работать (для регулятора)
- Reality защищает почти полностью (если конфиг правильный).
- Self-Steal требует, чтобы основной сайт **выглядел реальным** (не пустая страница).
- **Behavioral analysis post-probe:** DPI может отслеживать не только response на probe, но и **behavior после**. Если после probing'a IP пересталь принимать запросы (rate-limiting) — подозрительно.

## Минимальный сценарий проверки своего сервера
```bash
# С другого VPS (имитация ТСПУ-probe):
curl -v https://my-vpn-server.com:443/  # GET / — что отдаёт?
curl https://my-vpn-server.com:443/random-path  # 404? 200?
echo -e "GET / HTTP/1.0\r\n\r\n" | nc my-vpn-server.com 443  # raw socket
```

Если на любой из этих проб сервер отвечает «как VPN» (длинный delay, странный TLS-cert, no fallback) → уязвимо. Должно отвечать как обычный TLS-сайт.

## Связи
- **Базируется на:** [[ТСПУ]] (executor), концептуально — [[Виды атак]] (атака на VPN-сервер).
- **Используется против:** VPN-серверов всех типов.
- **Защищается через:** [[VLESS-Reality]] (target-fallback), [[Self-Steal — свой домен]] (real site), [[uTLS]] (handshake mimicry).
- **Соседи по уровню:** [[Session freezing]] (другая DPI-стратегия).

## Источники
- src-08, src-10.
