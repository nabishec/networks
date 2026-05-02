---
title: gRPC
aliases: ["gRPC", "gRPC Remote Procedure Call"]
type: protocol
layer: application
chapter: 7
difficulty: intermediate
prerequisites: ["[[RPC]]", "[[HTTP-2 и HTTP-3|HTTP/2 и HTTP/3]]"]
related: ["[[RPC]]", "[[HTTP]]", "[[Сжатие заголовков]]"]
tags: [networking, ch07]
---
# gRPC (gRPC Remote Procedure Call)

## TL;DR
Современный фреймворк удалённого вызова процедур от Google (open-source с 2015 г.). Транспорт — **HTTP/2** (мультиплексирование, header compression); сериализация — **Protocol Buffers** (бинарный, компактный, schema-first). Поддерживает **четыре модели** взаимодействия, включая двунаправленный стриминг. Стандарт de facto для межсервисного взаимодействия в микросервисной архитектуре. Не работает в браузере «как есть» — нужен **gRPC-Web** или прокси.

В книге Tanenbaum 6e отдельный раздел про gRPC отсутствует; материал ниже собран из открытых статей на Хабре (см. «Источники»).

## Какую проблему решает
Классический REST на HTTP/1.1 с JSON:
- **избыточен** в передаче (JSON-текст vs бинарный protobuf — в 3–10× больше байт),
- **медленный** на новых соединениях (HTTP/1.1 head-of-line blocking),
- **слабо типизирован** — контракт описан в OpenAPI, но «истина» в коде.

gRPC даёт строгий контракт через `.proto`-файл, генерацию stub'ов и двунаправленный стриминг. Идеально для service-to-service-вызовов в микросервисах с высокими требованиями к latency и throughput.

## Как работает

### Контракт через Protocol Buffers
В `.proto` описываются messages и services. Пример:

```protobuf
syntax = "proto3";

message UserRequest { int64 id = 1; }
message UserReply   { string name = 1; int32 age = 2; }

service UserService {
  rpc GetUser   (UserRequest) returns (UserReply);
  rpc StreamFeed(UserRequest) returns (stream FeedItem);
}
```

`protoc`-компилятор генерирует **клиентский и серверный stub** на нужном языке (Go, Java, Python, C++, Rust, ...) — приложение работает с готовыми типами и интерфейсами.

### Транспорт — HTTP/2
- Один TCP-коннект → много логических stream'ов одновременно (мультиплексирование).
- HPACK-сжатие заголовков (значимо для частых RPC с метаданными).
- Двунаправленные потоки на уровне frame'ов — естественная база для стриминга.
- gRPC-вызов — это HTTP/2 POST на `/<package>.<service>/<method>` с `Content-Type: application/grpc+proto`.

### 4 модели вызовов
| Модель | Запрос | Ответ | Пример |
|---|---|---|---|
| **Unary** | 1 | 1 | `GetUser(id)` → `User` |
| **Server streaming** | 1 | поток | подписка на ленту, отдача файла по chunks |
| **Client streaming** | поток | 1 | загрузка файла из чанков с финальным ACK |
| **Bidirectional streaming** | поток | поток | чат, real-time-телеметрия, видеозвонок-сигналинг |

REST в строгом смысле умеет только первую (request-response).

### Сравнение с REST
| | REST | gRPC |
|---|---|---|
| Транспорт | HTTP/1.1 | HTTP/2 (требуется на инфре) |
| Формат | JSON / XML | protobuf бинарный |
| Контракт | OpenAPI (опционально) | `.proto` обязателен |
| Стриминг | через WebSocket / SSE отдельно | 4 встроенные модели |
| Браузер | напрямую | через **gRPC-Web** + прокси |
| Размер payload | большой | в разы меньше |

(Сводная таблица — по [habr.com/ru/articles/953694](https://habr.com/ru/articles/953694/), [habr.com/ru/companies/otus/articles/780720](https://habr.com/ru/companies/otus/articles/780720/).)

## Пример
**Микросервис на Go** (server-streaming):
```go
func (s *Server) StreamFeed(req *pb.UserRequest, stream pb.UserService_StreamFeedServer) error {
    for _, item := range loadFeed(req.Id) {
        if err := stream.Send(item); err != nil { return err }
    }
    return nil
}
```
**Клиент:**
```go
client := pb.NewUserServiceClient(conn)
stream, _ := client.StreamFeed(ctx, &pb.UserRequest{Id: 42})
for {
    item, err := stream.Recv()
    if err == io.EOF { break }
    process(item)
}
```

В Яндексе gRPC массово используется как протокол межсервисного взаимодействия (см. доклад на [habr.com/ru/companies/yandex/articles/484068](https://habr.com/ru/companies/yandex/articles/484068/)). Также активно применяется для интеграции Spring Boot-микросервисов ([habr.com/ru/articles/910092](https://habr.com/ru/articles/910092/)) и при связке с Kafka в event-driven-архитектурах ([habr.com/ru/companies/ruvds/articles/912502](https://habr.com/ru/companies/ruvds/articles/912502/)).

## Связи
- **Базируется на:** [[RPC]] (концепт удалённого вызова), [[HTTP-2 и HTTP-3|HTTP/2]] (транспорт), Protocol Buffers (сериализация).
- **Используется в:** микросервисной архитектуре, межсервисной коммуникации, IoT (минимизация трафика), мобильных backend'ах.
- **Соседи по уровню:** REST/JSON, GraphQL, Apache Thrift (старший родственник от Facebook), Avro (Hadoop-стек), Cap'n Proto.
- **Противопоставляется:** REST — простота и универсальность браузеров; gRPC — производительность и строгая типизация.

## Подводные камни
- **Браузер из коробки не умеет gRPC** — нужен **gRPC-Web** + прокси (Envoy / grpcwebproxy). Передавать gRPC прямо в браузер пробовали неоднократно (см. [habr.com/ru/articles/944894](https://habr.com/ru/articles/944894/)).
- **HTTP/2-инфраструктура** обязательна end-to-end: corporate-прокси, балансировщики, IDS должны корректно обрабатывать H2.
- **Балансировка нагрузки** на L4 (TCP-LB) ломает мультиплексирование gRPC — все RPC сессии одного клиента липнут к одному backend'у. Нужны L7-балансировщики (Envoy, Linkerd) или client-side LB.
- **Версионирование** протобафа: добавлять поля можно (нумерация), удалять — осторожно (помечать `reserved`); смена типа поля — breaking change.
- **Дебаг сложнее REST'а:** бинарный формат, не открыть в curl. Решения — `grpcurl`, BloomRPC, Postman gRPC-mode.
- **Deadline/cancellation** — сильная сторона: контекст с дедлайном пробрасывается насквозь по цепочке вызовов и автоматически отменяет все ниже.

## Источники / Дальше читать
- [[RPC]] — общая модель.
- [[HTTP-2 и HTTP-3]] — транспорт.
- Habr: [Введение в gRPC: Основы, применение, плюсы и минусы](https://habr.com/ru/articles/819821/) — June 2024.
- Habr: [gRPC и Protocol Buffers: современный подход к обмену данными](https://habr.com/ru/companies/otus/articles/780720/) — Sept 2025.
- Habr: [gRPC для тестировщика: быстрый старт после REST](https://habr.com/ru/articles/953694/) — Oct 2025 (детально про 4 модели вызовов).
- Habr: [gRPC в качестве протокола межсервисного взаимодействия — доклад Яндекса](https://habr.com/ru/companies/yandex/articles/484068/).
- Tanenbaum, гл. 6, §6.4.2 — общая модель RPC, отдельной главы про gRPC в книге нет.
