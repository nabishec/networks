---
title: HLS и DASH
aliases: ["HTTP Live Streaming", "MPEG-DASH", "Adaptive bitrate streaming"]
type: protocol
layer: application
chapter: 7
difficulty: basic
prerequisites: ["[[HTTP]]", "[[Цифровое видео — кодеки]]"]
related: ["[[CDN — устройство]]"]
tags: [networking, ch07]
---
# HLS и DASH — Adaptive Bitrate Streaming

## TL;DR
**HLS** (HTTP Live Streaming, Apple, RFC 8216) и **MPEG-DASH** (Dynamic Adaptive Streaming over HTTP, ISO 23009-1) — стандарты потоковой видеопередачи через **обычный HTTP**. Видео нарезается на короткие **сегменты** (2-10 с) в нескольких качествах (480p/720p/1080p/4K). **Manifest** (m3u8 или MPD) описывает сегменты. Клиент сам выбирает нужное качество исходя из своей пропускной способности → **adaptive**. Доминируют в YouTube, Netflix, Twitch.

## Какую проблему решает
**Старый streaming** (RTMP, RTSP) использовал custom-протоколы → требовал особых серверов, проблемы с firewall'ами/NAT'ом. **HLS/DASH** просто **HTTP** → работает где угодно (CDN, кэш, прокси), легко масштабируется. Адаптивный битрейт даёт **отзывчивость**: загруженный тоннель → клиент сам перейдёт на 480p.

## Как работает

**Подготовка контента:**
1. Видео кодируется в **несколько качеств** (renditions): 240p / 480p / 720p / 1080p / 4K.
2. Каждое качество нарезается на **сегменты** (2-10 секунд).
3. **Manifest-файл** описывает renditions и сегменты.

**HLS manifest** (m3u8, plain text):
```
#EXTM3U
#EXT-X-STREAM-INF:BANDWIDTH=400000,RESOLUTION=480x270
low/index.m3u8
#EXT-X-STREAM-INF:BANDWIDTH=2000000,RESOLUTION=1280x720
high/index.m3u8
```

Каждый rendition — собственный m3u8:
```
#EXTM3U
#EXT-X-TARGETDURATION:6
#EXTINF:6.0,
seg001.ts
#EXTINF:6.0,
seg002.ts
```

**DASH manifest (MPD)** — XML с похожей структурой.

**Player loop:**
1. Получает manifest.
2. Замеряет свою пропускную способность.
3. Выбирает rendition.
4. Скачивает сегменты по очереди (ahead-of-playback buffer).
5. **На каждой сегменте** перепроверяет — переключается на другой rendition при необходимости.

**Live streaming:** manifest обновляется регулярно, добавляются новые сегменты.

## Пример
**YouTube Live:**
- Стример → encoder → сегменты H.264 в нескольких качествах + Opus аудио.
- Сегменты + manifest на CDN.
- Зрители скачивают сегменты по HTTP.
- При плохом сигнале player переключает на 480p.

**Latency:**
- HLS classic: ~30 секунд (3-4 segments × 6 сек buffer).
- LL-HLS (Low-Latency, 2019, доработано 2020): ~2-5 секунд через partial segments (CMAF chunks) + blocking playlist reload + preload hints (HTTP/2 push был исключён в 2020).
- DASH-LL аналогично.

## Связи
- **Базируется на:** [[HTTP]] (transport), [[Цифровое видео — кодеки]], [[CDN — устройство]].
- **Используется в:** YouTube, Netflix, Twitch, Apple TV+, all major streaming.
- **Соседи по уровню:** **WebRTC** — для real-time двустороннего видео; **RTMP** — устаревший legacy ingest.
- **Противопоставляется:** UDP-based streaming (MPEG-TS over UDP) — для broadcast; HLS/DASH — для VOD/live над интернетом.

## Подводные камни
- **Buffer vs latency:** больше buffer → меньше пропусков, больше задержка. Live и VOD требуют разных настроек.
- **Manifest не должен быть слишком long** — клиент тратит время на парсинг. Современные потоки используют **multi-period** для разделения.
- **Encryption (DRM):** Widevine, FairPlay, PlayReady — обычно поверх HLS/DASH. Лицензии на сегменты.

## Дальше читать
- [[CDN — устройство]] — где живут сегменты.
- [[Цифровое видео — кодеки]] — что внутри сегмента.
- Tanenbaum, гл. 7, §7.4.3 (стр. PDF 766–774).
