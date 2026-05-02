---
title: План Фазы B — концепты по главам
type: meta
tags: [meta, plan]
---
# План Фазы B — концепты по главам

Источник: Tanenbaum, Feamster, Wetherall. «Компьютерные сети», 6-е изд. (Питер, 2023).
Формат: `[[имя]] (папка) — аннотация в одну строку.` Папки: `concepts/protocols/algorithms` (10/20/30).

## Глава 1. Введение (стр. 26–120)

- [[Компьютерная сеть]] (concepts) — определение, отличие от распределённой системы.
- [[Хост]] (concepts) — конечный узел, генерирующий/потребляющий трафик.
- [[Маршрутизатор]] (concepts) — узел, пересылающий пакеты между сетями.
- [[Топология сети]] (concepts) — связи узлов: шина, звезда, кольцо, mesh.
- [[Типы сетей по охвату]] (concepts) — PAN, LAN, MAN, WAN, internetwork.
- [[Сети широкополосного доступа]] (concepts) — DSL, кабель, оптоволокно «последней мили».
- [[CDN — сеть доставки контента]] (concepts) — географически распределённое кэширование.
- [[Транзитная сеть]] (concepts) — IP transit, ISP-tier-1/2/3, пиринг.
- [[Корпоративная сеть]] (concepts) — внутренняя сеть организации, VLAN, NAC.
- [[Интернет — архитектура]] (concepts) — autonomous systems, IXP, BGP-связность.
- [[Wi-Fi — обзор]] (concepts) — IEEE 802.11, точка доступа, ассоциация.
- [[Сотовая сеть — обзор]] (concepts) — соты, базовая станция, handoff.
- [[Цели проектирования сетей]] (concepts) — надёжность, масштабируемость, безопасность, эволюция.
- [[Уровневая архитектура]] (concepts) — protocol stack, инкапсуляция, разделение ответственности.
- [[Протокол vs служба]] (concepts) — service primitives, разница интерфейса и реализации.
- [[Соединение vs без соединения]] (concepts) — connection-oriented (вирт. канал) vs connectionless (датаграмма).
- [[Надёжность службы]] (concepts) — гарантии доставки, порядок, дубли.
- [[Примитив службы]] (concepts) — CONNECT/SEND/RECEIVE/DISCONNECT, синхронность.
- [[Эталонная модель OSI]] (concepts) — 7 уровней, теоретический эталон.
- [[Эталонная модель TCP/IP]] (concepts) — 4 уровня, де-факто стандарт интернета.
- [[Гибридная модель Tanenbaum]] (concepts) — 5 уровней, используется в книге.
- [[Стандартизация сетей]] (concepts) — IEEE/ISO/ITU/IETF/IAB, RFC.
- [[Сетевой нейтралитет]] (concepts) — равноправная обработка трафика, политика ISP.
- [[Единицы измерения скорости]] (concepts) — бит/с vs байт/с, степени 10 vs 2.

## Глава 2. Физический уровень (стр. 121–241)

- [[Среда передачи данных]] (concepts) — ограничивает пропускную способность и расстояние.
- [[Витая пара]] (concepts) — UTP/STP, категории Cat-5/6/7, экранирование.
- [[Коаксиальный кабель]] (concepts) — внутренний/внешний проводник, исторически — Ethernet и кабельное ТВ.
- [[Оптоволокно]] (concepts) — одномодовое vs многомодовое, потери, окна 1310/1550 нм.
- [[Спектр электромагнитных волн]] (concepts) — диапазоны, лицензируемые vs нелицензируемые.
- [[Расширение спектра — FHSS]] (algorithms) — Frequency Hopping, устойчивость к узкополосной помехе.
- [[Расширение спектра — DSSS]] (algorithms) — Direct Sequence, чип-последовательность, gain.
- [[Сверхширокополосная связь]] (concepts) — UWB, импульсы наносекундной длительности.
- [[Теорема Найквиста]] (concepts) — макс. скорость в шумосвободном канале B·log2(V).
- [[Закон Шеннона]] (concepts) — предел канала с шумом B·log2(1+S/N).
- [[Цифровая модуляция — линейные коды]] (concepts) — NRZ, NRZI, Manchester, 4B/5B.
- [[Цифровая модуляция — амплитуда/частота/фаза]] (concepts) — ASK, FSK, PSK, QAM, диаграмма созвездия.
- [[Мультиплексирование]] (concepts) — TDM, FDM, OFDM, WDM, CDMA — разделение канала.
- [[OFDM]] (concepts) — orthogonal frequency division multiplexing, основа Wi-Fi/LTE.
- [[Коммутация каналов vs пакетов]] (concepts) — фундаментальный архитектурный выбор.
- [[PSTN — телефонная сеть]] (concepts) — структура: local loop, end office, toll office.
- [[Местная петля]] (concepts) — медный провод от АТС до абонента.
- [[ADSL]] (protocols) — асимметричный xDSL поверх местной петли.
- [[Сотовая сеть — соты и handoff]] (concepts) — реиспользование частот, переход между сотами.
- [[GSM]] (protocols) — 2G, TDMA+FDMA, SIM, BSS/NSS.
- [[Поколения сотовой связи 1G–5G]] (concepts) — эволюция: голос → данные → IoT → URLLC.
- [[HFC — гибридная сеть]] (concepts) — оптика до узла, коаксиал до дома.
- [[DOCSIS]] (protocols) — стандарт передачи данных по кабельному ТВ.
- [[Спутники GEO/MEO/LEO]] (concepts) — геостационарные/средние/низкие орбиты.

## Глава 3. Канальный уровень (стр. 242–311)

- [[Канальный уровень]] (concepts) — задачи: framing, error/flow control, надёжная доставка по каналу.
- [[Фрейм]] (concepts) — единица канального уровня, границы определены framing-схемой.
- [[Формирование фреймов]] (concepts) — методы: подсчёт байтов, флаг+стаффинг, физ. сигнал.
- [[Битстаффинг]] (algorithms) — вставка нулей в HDLC для защиты флаговой последовательности.
- [[Корректирующие коды — Хэмминг]] (algorithms) — исправление 1 ошибки, расстояние Хэмминга.
- [[Код Рида-Соломона]] (algorithms) — пакетные ошибки, основа CD/DVD/QR.
- [[LDPC и сверточные коды]] (algorithms) — современные FEC для беспроводных и хранилищ.
- [[CRC — циклический избыточный код]] (algorithms) — детектирование ошибок через полином-генератор.
- [[Internet checksum]] (algorithms) — простая контрольная сумма IP/TCP/UDP, 16-битное сложение.
- [[Stop-and-Wait]] (algorithms) — простейший ARQ: один фрейм, ждём ACK.
- [[Скользящее окно]] (concepts) — много фреймов в полёте, окно отправителя/получателя.
- [[Go-Back-N]] (algorithms) — окно отправителя > 1, получателя = 1, повтор с N.
- [[Selective Repeat]] (algorithms) — оба окна > 1, повтор только потерянного.
- [[Piggybacking]] (concepts) — встраивание ACK в обратный data-фрейм.
- [[ARQ]] (concepts) — Automatic Repeat reQuest — общий принцип повтора при ошибках.
- [[PPP]] (protocols) — Point-to-Point Protocol — каналы SONET, dial-up, эмуляция Ethernet.
- [[SONET]] (concepts) — синхронная оптическая сеть, иерархия STS-1/3/12/192.

## Глава 4. MAC-подуровень (стр. 312–411)

- [[Подуровень MAC]] (concepts) — арбитраж разделяемой среды, отдельно от LLC.
- [[Статическое vs динамическое распределение канала]] (concepts) — TDMA/FDMA vs ALOHA/CSMA.
- [[ALOHA]] (algorithms) — pure/slotted, первый протокол случайного доступа.
- [[CSMA]] (algorithms) — Carrier Sense Multiple Access: 1-persistent, non-persistent, p-persistent.
- [[CSMA/CD]] (algorithms) — обнаружение коллизий, основа классического Ethernet.
- [[CSMA/CA]] (algorithms) — избегание коллизий, основа Wi-Fi (DCF).
- [[Bit-map и token-passing]] (algorithms) — безконфликтные протоколы, FDDI/Token Ring.
- [[Hidden terminal problem]] (concepts) — узлы вне зоны слышимости друг друга → коллизия у AP.
- [[Exposed terminal problem]] (concepts) — необоснованное молчание из-за чужой передачи.
- [[Ethernet — IEEE 802.3]] (protocols) — фрейм, MAC-адрес, EtherType.
- [[MAC-адрес]] (concepts) — 48-бит, OUI+NIC, локальный/универсальный, уникаст/мультикаст.
- [[Манчестерское кодирование]] (algorithms) — самосинхронизирующийся код классич. Ethernet.
- [[Коммутируемый Ethernet]] (concepts) — switch вместо концентратора, отдельный домен коллизий.
- [[Эволюция Ethernet]] (concepts) — Fast/Gigabit/10G/40G/100G — что меняется и что остаётся.
- [[802.11 — Wi-Fi архитектура]] (protocols) — STA, AP, BSS, ESS, infrastructure/ad-hoc.
- [[802.11 MAC — DCF]] (algorithms) — Distributed Coordination Function на CSMA/CA.
- [[RTS/CTS]] (algorithms) — Request/Clear-to-Send против hidden terminal.
- [[NAV — Network Allocation Vector]] (concepts) — виртуальный carrier sensing в 802.11.
- [[Bluetooth]] (protocols) — piconet/scatternet, master/slave, profiles, BLE.
- [[Мост и обучающийся мост]] (concepts) — bridge, MAC learning table, разделение коллизионных доменов.
- [[Spanning Tree Protocol]] (algorithms) — STP, root bridge, устранение петель.
- [[Hub vs Switch vs Router vs Gateway]] (concepts) — на каком уровне работает каждый.
- [[VLAN — IEEE 802.1Q]] (concepts) — виртуальные LAN, тегирование, trunk.

## Глава 5. Сетевой уровень (стр. 412–562)

- [[Сетевой уровень]] (concepts) — маршрутизация end-to-end через возможно много сетей.
- [[Datagram subnet vs virtual-circuit subnet]] (concepts) — IP vs ATM/MPLS, без/с сост. соед.
- [[Принцип оптимальности маршрутизации]] (concepts) — sink tree, Bellman'овская декомпозиция.
- [[Алгоритм Дейкстры]] (algorithms) — shortest path, неотриц. веса, основа OSPF.
- [[Лавинная маршрутизация (flooding)]] (algorithms) — широковещание + защита от петель.
- [[Distance Vector Routing]] (algorithms) — обмен таблицами с соседями, основа RIP.
- [[Count-to-infinity problem]] (concepts) — болезнь distance vector при разрыве линии.
- [[Link State Routing]] (algorithms) — flood топологии, локально вычисляем Дейкстру.
- [[Иерархическая маршрутизация]] (concepts) — area/AS, ограничение размера таблиц.
- [[Multicast routing]] (concepts) — IGMP, PIM-DM/SM, RPF, дерево распределения.
- [[Anycast]] (concepts) — один адрес — много экземпляров, маршрут к ближайшему.
- [[Перегрузка сети]] (concepts) — congestion ≠ link-saturation, точка коллапса.
- [[Token bucket]] (algorithms) — ограничитель скорости с burst.
- [[Leaky bucket]] (algorithms) — сглаживатель трафика без burst.
- [[RED и AQM]] (algorithms) — Random Early Detection, активное управление очередью.
- [[ECN]] (concepts) — Explicit Congestion Notification, alternative loss.
- [[QoS vs QoE]] (concepts) — измеримые параметры vs воспринимаемое качество.
- [[IntServ и RSVP]] (concepts) — резервирование ресурсов per-flow, не масштабируется.
- [[DiffServ]] (concepts) — классы трафика DSCP, per-hop behavior, основной механизм QoS в ядре.
- [[Internetworking — фрагментация]] (concepts) — разные MTU, фрагментация и PMTUD.
- [[Туннелирование]] (concepts) — инкапсуляция IPv6 в IPv4, GRE, IPsec-туннель.
- [[SDN — программно-конфигурируемые сети]] (concepts) — control plane отделён от data plane.
- [[OpenFlow]] (protocols) — протокол между SDN-контроллером и коммутатором.
- [[IPv4]] (protocols) — заголовок, адресация, опции, fragmentation flags.
- [[IP-адресация и CIDR]] (concepts) — классовая → бесклассовая, prefix length.
- [[NAT]] (concepts) — Network Address Translation, преодоление дефицита IPv4.
- [[IPv6]] (protocols) — 128 бит, упрощённый заголовок, SLAAC, нет фрагментации в маршрутизаторах.
- [[ICMP]] (protocols) — служебные сообщения IP: echo, unreachable, time exceeded.
- [[ARP]] (protocols) — Address Resolution Protocol, IP→MAC в LAN.
- [[DHCP]] (protocols) — Dynamic Host Configuration Protocol, автоконфигурация хоста.
- [[MPLS]] (protocols) — Multi-Protocol Label Switching, label switching поверх IP.
- [[OSPF]] (protocols) — link-state IGP, area, LSA, Дейкстра.
- [[BGP]] (protocols) — path-vector EGP, политика, AS-path, no-export, MED.

## Глава 6. Транспортный уровень (стр. 563–683)

- [[Транспортный уровень]] (concepts) — end-to-end, надёжность поверх ненадёжной сети.
- [[Сокеты Беркли]] (concepts) — socket(), bind(), listen(), accept(), connect(), API.
- [[Порт]] (concepts) — 16-бит идентификатор приложения, well-known/registered/dynamic.
- [[Three-way handshake]] (algorithms) — SYN, SYN-ACK, ACK; защита от старых дублей.
- [[Разрыв соединения]] (concepts) — FIN, RST; полузакрытое состояние.
- [[Управление потоком vs контроль перегрузки]] (concepts) — flow control vs congestion control, разные цели.
- [[AIMD]] (algorithms) — Additive Increase Multiplicative Decrease, fairness и стабильность.
- [[UDP]] (protocols) — без соед., 8-байтный заголовок, контрольная сумма.
- [[RPC — удалённый вызов процедур]] (concepts) — модель «локального вызова», маршалинг, идемпотентность.
- [[RTP]] (protocols) — Real-time Transport Protocol, медиа поверх UDP, sequence/timestamp.
- [[TCP]] (protocols) — байтовый поток, надёжность, упорядочивание, контроль потока и перегрузки.
- [[TCP — заголовок]] (concepts) — порты, seq, ack, флаги SYN/ACK/FIN/RST/PSH/URG, window.
- [[TCP — состояния]] (concepts) — конечный автомат: LISTEN, SYN_SENT, ESTABLISHED, TIME_WAIT.
- [[TCP — раздвижное окно]] (concepts) — flow control через rwnd, Nagle, silly window syndrome.
- [[Karn algorithm]] (algorithms) — измерение RTT при ретрансмиссиях.
- [[TCP — slow start]] (algorithms) — экспонент. рост cwnd до ssthresh.
- [[TCP — congestion avoidance]] (algorithms) — линейный рост cwnd после ssthresh, fast retransmit, fast recovery.
- [[CUBIC TCP]] (algorithms) — кубический рост cwnd, основной алгоритм Linux.
- [[BBR]] (algorithms) — model-based congestion control, оценка bottleneck BW и min RTT.
- [[QUIC]] (protocols) — UDP-based, шифрование, мультиплексирование без HoL blocking.
- [[Сжатие заголовков]] (concepts) — ROHC, HPACK, QPACK — экономия в мобильных сетях.

## Глава 7. Прикладной уровень (стр. 684–812)

- [[DNS]] (protocols) — иерархическое именование, разрешение в IP.
- [[DNS — типы записей]] (concepts) — A, AAAA, MX, CNAME, NS, TXT, SOA, PTR.
- [[DNS — рекурсивный vs итеративный запрос]] (concepts) — резолвер vs корневой/TLD/auth NS.
- [[DNS — кэширование и TTL]] (concepts) — снижение нагрузки, конфликт со свежестью.
- [[DNS over HTTPS / TLS]] (protocols) — DoH/DoT — конфиденциальность DNS.
- [[Email — архитектура]] (concepts) — MUA, MSA, MTA, MDA — кто кому передаёт.
- [[SMTP]] (protocols) — Simple Mail Transfer Protocol, port 25/587, EHLO/MAIL FROM/RCPT TO/DATA.
- [[IMAP и POP3]] (protocols) — доставка с сервера к клиенту, разница моделей.
- [[MIME]] (concepts) — Multipurpose Internet Mail Extensions: типы, кодировки, base64.
- [[Web — архитектура]] (concepts) — клиент, сервер, прокси, URL.
- [[HTTP]] (protocols) — методы, статус-коды, headers, persistent connections.
- [[HTTP/2 и HTTP/3]] (protocols) — мультиплексирование, бинарный фрейминг, поверх QUIC.
- [[HTTPS]] (protocols) — HTTP поверх TLS, цепочка сертификатов.
- [[Cookies и web tracking]] (concepts) — Set-Cookie, third-party, fingerprinting.
- [[Веб-кэширование и прокси]] (concepts) — Cache-Control, ETag, прозрачные/обратные прокси.
- [[Динамические веб-приложения]] (concepts) — server-side, AJAX, SPA, REST.
- [[Цифровой звук — кодеки]] (concepts) — PCM, MP3, AAC, Opus — разница и применение.
- [[Цифровое видео — кодеки]] (concepts) — MPEG-2/H.264/H.265/AV1, I/P/B-кадры.
- [[HLS и DASH]] (protocols) — HTTP Live Streaming, MPEG-DASH, адаптивный битрейт.
- [[VoIP и SIP]] (protocols) — голос поверх IP, SIP-сигнализация, RTP-медиа.
- [[CDN — устройство]] (concepts) — edge-серверы, anycast, GSLB, request routing.
- [[P2P-сети]] (concepts) — BitTorrent, DHT, Chord, Kademlia.

## Глава 8. Сетевая безопасность (стр. 813–959)

- [[Триада CIA]] (concepts) — Confidentiality, Integrity, Availability — основа модели угроз.
- [[Виды атак]] (concepts) — reconnaissance, eavesdropping, spoofing, DoS/DDoS.
- [[ARP spoofing]] (concepts) — отравление ARP-кэша, MITM в LAN.
- [[DNS spoofing и cache poisoning]] (concepts) — подмена ответа, защита через DNSSEC.
- [[Брандмауэр]] (concepts) — packet filter, stateful, application gateway.
- [[IDS и IPS]] (concepts) — сигнатуры/аномалии, обнаружение vs предотвращение.
- [[Симметричная vs асимметричная криптография]] (concepts) — общий ключ vs пара ключей.
- [[Одноразовый блокнот]] (concepts) — one-time pad, абсолютная стойкость, проблема ключа.
- [[DES и AES]] (algorithms) — блочные шифры с симм. ключом, эволюция стандартов.
- [[Режимы шифрования]] (concepts) — ECB (плохо), CBC, CTR, GCM (AEAD).
- [[RSA]] (algorithms) — асимм. шифрование на факторизации больших чисел.
- [[Диффи-Хеллман]] (algorithms) — обмен ключами на дискретном логарифме.
- [[Цифровая подпись]] (concepts) — невозможность отказа, integrity, аутентичность.
- [[Хеш-функции]] (concepts) — SHA-2/3, свойства: одностор., collision-resistance.
- [[HMAC]] (algorithms) — keyed-hash MAC, защита целостности и аутентичности.
- [[Birthday attack]] (concepts) — на коллизии хэша, влияет на длину выхода.
- [[X.509 сертификаты]] (concepts) — структура, цепочка, OID, расширения.
- [[PKI и центры сертификации]] (concepts) — иерархия CA, отзыв (CRL/OCSP).
- [[Kerberos]] (protocols) — KDC, ticket, аутентификация в Windows-домене.
- [[IPsec]] (protocols) — AH, ESP, IKE; transport vs tunnel mode.
- [[VPN]] (concepts) — site-to-site/remote access, поверх IPsec/TLS/WireGuard.
- [[WPA2 и WPA3]] (protocols) — Wi-Fi security, 4-way handshake, SAE.
- [[PGP и S/MIME]] (protocols) — end-to-end шифрование почты.
- [[TLS — рукопожатие]] (protocols) — ClientHello, обмен ключами, аутентификация сервера.
- [[Tor — анонимная связь]] (concepts) — onion routing, 3 узла, exit relay.

---

## Замечание

Это срез на ~200 концептов; глава 5 и 8 идут на верхнем пределе (~33 и ~25). При построении заметок отдельные концепты могут быть объединены или разделены — план носит ориентировочный характер.

При первом упоминании англ. термина — даём его в скобках. Каждое имя из плана = заголовок будущей заметки = wikilink в других заметках.
