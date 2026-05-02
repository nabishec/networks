---
title: Yandex API Gateway фронтинг
aliases: ["Yandex Cloud Functions", "API Gateway proxy"]
type: technique
layer: application
status: working
status_as_of: 2026-05-02
risk: high
prerequisites: ["[[CDN-фронтинг]]", "[[HTTPS]]"]
related: ["[[VLESS-Reality]]", "[[Self-Steal — свой домен]]"]
sources: [src-01, src-06]
tags: [networking, rf-circumvention, technique]
---
# Yandex API Gateway фронтинг

## TL;DR
**Внутрь-РФ-фронтинг** через инфраструктуру Yandex Cloud. API Gateway или Cloud Functions выступают как **разрешённый IP** (Yandex AS попадает в whitelist оператора), а проксируют запросы наружу к скрытому backend. Аналогично работают **VK Cloud** и **DiNet**. Высокий риск с правовой точки зрения — использование российских облаков для обхода может быть отслежено провайдером (но публикация платежей — отдельная история).

## Какую проблему решает
**Мобильные whitelist-операторы** (src-01, src-05) пропускают только сервисы из узкого whitelist. Yandex (AS13238) — там. Cloudflare/AWS/etc — может не быть. Поэтому **первый hop через Yandex** проходит, а дальше Yandex сам идёт куда хочет (за пределы РФ-whitelist'а — Yandex транзитный AS).

## Как работает

**Архитектура (по src-01, PB1):**
```
Client (mobile, whitelist-RU) → DNS resolve proxy.example.com →
Cloudflare (резолв в CNAME) → Yandex API Gateway IP →
API Gateway forward → backend (любой VPS) → Internet
```

**Точные шаги:**
1. Купить домен.
2. Перевести DNS на Cloudflare.
3. Получить Let's Encrypt-сертификат через Yandex Certificate Manager.
4. Создать API Gateway в Yandex Cloud с OpenAPI-спецификацией прокси:
   ```yaml
   openapi: 3.0.0
   paths:
     /{proxy+}:
       x-yc-apigateway-integration:
         type: http
         url: "https://my-backend.com/{proxy}"
   ```
5. Привязать домен и сертификат к шлюзу.
6. В Cloudflare добавить CNAME `proxy.example.com` → служебный домен Yandex API Gateway.
7. Клиент шлёт TLS-handshake с SNI=proxy.example.com → DNS даёт Yandex-IP → пропускается мобильным оператором → Yandex проксирует к backend.

**Cloud Functions:** alternativa — функция-прокси на Node.js/Python, которая `fetch`'ит к backend и возвращает.

## Где ломается / почему может не работать
- **Бесплатный/cheap-tier Yandex Cloud** имеет лимиты (трафик, RPS) — для VPN мощно нагружает.
- **Yandex может детектировать** abuse-pattern (long-lived connections через API Gateway) и заблокировать аккаунт.
- **Цена при большом трафике** растёт.
- Если **РКН включит Yandex AS в whitelist более жёстко** (точечно по доменам внутри Yandex) — техника сломается.
- **Личные данные:** оплата с российской карты + привязка к Яндекс-аккаунту → высокая trace-ability.

## Минимальный пошаговый сценарий
См. шаги в «Как работает». Полный пример конфига — в [[PB1 — Yandex API Gateway фронтинг]].

## Что нужно
- Yandex Cloud аккаунт (с привязанной картой).
- Свой домен.
- Cloudflare-аккаунт (для DNS-делегирования).
- Backend VPS (вне РФ).
- Yandex CLI или Web-консоль для конфигурации.

## Связи
- **Базируется на:** [[CDN-фронтинг]] (концептуальный родитель), [[HTTPS]], [[X.509 сертификаты]].
- **Используется в:** [[PB1 — Yandex API Gateway фронтинг]], [[PB3 — 4-уровневая архитектура за 265₽]] (один уровень).
- **Соседи по уровню:** **VK Cloud** аналогичный, **DiNet** аналогично; [[CDN-фронтинг]] глобальный.
- **Противопоставляется:** прямое подключение к зарубежному VPS — заблокировано в whitelist-моделях.

## Источники
- src-01, src-06.
