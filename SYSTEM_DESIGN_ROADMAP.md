# System Design / Architecture — повний роадмап від джуна до сеньйора

> Структурований план вивчення архітектури розподілених систем: що, в якому порядку, з яких джерел, і як практикувати.

---

## Зміст

1. [Вступ — що таке System Design](#1-вступ)
2. [Передумови (foundations)](#2-передумови)
3. [Junior рівень](#3-junior-рівень)
4. [Middle рівень](#4-middle-рівень)
5. [Senior рівень](#5-senior-рівень)
6. [Staff/Principal рівень](#6-staffprincipal-рівень)
7. [Класичні задачі System Design](#7-класичні-задачі-system-design)
8. [Рамка відповіді на інтерв'ю](#8-рамка-відповіді-на-інтервю)
9. [Ресурси](#9-ресурси)
10. [Таймлайн вивчення](#10-таймлайн-вивчення)
11. [.NET-специфіка](#11-net-специфіка)

---

## 1. Вступ

### Що таке System Design

**System Design** — це дисципліна про те, як будувати програмні системи, які:
- Масштабуються (scalability)
- Не падають (reliability)
- Швидко відповідають (performance)
- Легко підтримуються (maintainability)
- Вміщаються в бюджет і команду (pragmatic trade-offs)

На відміну від алгоритмічного інтерв'ю (де є одна правильна відповідь), system design — це **відкрита розмова про trade-offs**. Немає "правильної" архітектури — є компроміси.

### Коли починати вивчати

- **Джуни:** основи (REST, SQL, кешування, monolith) — з самого початку
- **Middle-рівень:** серйозно заглиблюватись у розподілені системи після 2-3 років досвіду
- **Senior-інтерв'ю:** обов'язково готуватись мінімум 2-3 місяці, навіть якщо вже маєш досвід

### Чому це важливо

1. **Інтерв'ю.** Для senior-позицій у FAANG/топ-компаніях system design — 40-60% фінальної оцінки.
2. **Робота.** Від Middle+ тебе вже чекають архітектурні рішення, не просто кодинг.
3. **Розуміння.** Дає картину "як насправді працюють великі системи" — Twitter, Netflix, Amazon.

---

## 2. Передумови

Перш ніж занурюватись у System Design, потрібне фундаментальне розуміння:

### 2.1 Комп'ютерні мережі

**Мінімум:**
- OSI model (7 рівнів) і TCP/IP model (4 рівні)
- TCP vs UDP — коли яке використовувати
- HTTP/1.1, HTTP/2, HTTP/3 (QUIC) — різниці
- HTTPS і TLS handshake
- DNS — як працює резолв імен
- CDN — як і навіщо
- WebSockets, Server-Sent Events, Long Polling
- Проксі vs Reverse Proxy

**Практика:** запустити nginx як reverse proxy, налаштувати TLS через Let's Encrypt, подебажити мережевий трафік через Wireshark.

### 2.2 Операційні системи

**Мінімум:**
- Процеси vs потоки (threads)
- Контекстний свіч, його вартість
- Пам'ять: stack vs heap, virtual memory, paging
- File descriptors, sockets
- I/O: блокуючий vs неблокуючий, sync vs async
- epoll/kqueue — як працює event loop

### 2.3 Бази даних — базис

- ACID властивості
- Індекси: B-tree, hash index, композитні
- Запити EXPLAIN — як читати план виконання
- Транзакції, рівні ізоляції (Read Uncommitted, Read Committed, Repeatable Read, Serializable)
- Foreign keys, constraints
- Normal forms (1NF, 2NF, 3NF) і коли денормалізувати

### 2.4 Структури даних для систем

З попереднього README алгоритмів, але з акцентом на застосування:
- Hash table — базис кешів, індексів, dedup
- B-tree — індекси в SQL БД
- LSM-tree — write-heavy БД (Cassandra, RocksDB, LevelDB)
- Bloom filter — швидкий pre-check перед дорогим lookup
- Trie — autocomplete, routing tables
- Merkle tree — синхронізація даних, blockchain
- Skip list — альтернатива B-tree (Redis sorted sets)
- HyperLogLog — кардинальність unique visitors
- Count-Min Sketch — approximate frequency counting

### 2.5 Базова математика для оцінок

Треба вміти робити back-of-the-envelope розрахунки:

| Величина | Значення |
|----------|----------|
| 1 KB | 10³ байт |
| 1 MB | 10⁶ байт |
| 1 GB | 10⁹ байт |
| 1 TB | 10¹² байт |
| 1 PB | 10¹⁵ байт |

**Стандартні затримки (2020s):**
| Операція | Час |
|----------|-----|
| L1 cache | 0.5 нс |
| L2 cache | 7 нс |
| RAM access | 100 нс |
| SSD read | 100 мкс |
| HDD seek | 10 мс |
| Round trip у дата-центрі | 0.5 мс |
| Round trip між регіонами | 50-150 мс |
| Packet CA → Європа → CA | ~150 мс |

**Оцінка навантаження:**
- Секунд на добу: ~86 400 ≈ 10⁵
- QPS = DAU × запитів_на_користувача / 86400
- Для read-heavy систем: пік = 2-3× середнього
- Ratio read/write зазвичай: 100:1 (соцмережа), 1000:1 (search engine)

---

## 3. Junior рівень

**Фокус:** зрозуміти одну-дві машини, базову веб-архітектуру. Ти маєш вміти пояснити, як працює "класична" вебзастосунок від браузера до БД.

### 3.1 Теми Junior рівня

#### Client-Server архітектура
- Як browser робить HTTP request
- Життєвий цикл запиту: DNS → TCP → TLS → HTTP → server → response
- Що таке stateless і чому REST стандартно stateless

#### REST API Design
- HTTP методи: GET, POST, PUT, PATCH, DELETE, HEAD, OPTIONS
- Status codes: 2xx/3xx/4xx/5xx — коли який
- Idempotency — чому PUT/DELETE idempotent, а POST — ні
- Версіонування API (URL path, header, query param)
- Pagination (offset vs cursor-based)
- REST vs RPC vs GraphQL — різниці

#### Monolith vs Microservices (базове розуміння)
- Коли monolith — нормально (99% стартапів)
- Коли варто дробити на сервіси
- Distributed monolith — антипатерн

#### SQL vs NoSQL — базові розрізнення
- Коли SQL (PostgreSQL, MySQL, SQL Server): транзакції, складні JOIN, консистентність
- Коли key-value (Redis, DynamoDB): кешування, сесії, швидкий lookup
- Коли документи (MongoDB): нечіткі схеми, вкладені структури
- Коли колонки (Cassandra, ScyllaDB): write-heavy, time-series
- Коли граф (Neo4j): соцмережі, рекомендації

#### Кешування (базове)
- In-process cache vs shared cache (Redis/Memcached)
- Cache hit/miss ratio
- TTL — коли використовувати
- Стратегії: cache-aside (найпопулярніша)

#### Load Balancing (концепт)
- Навіщо потрібен балансувальник
- Round robin, least connections, IP hash
- L4 (TCP) vs L7 (HTTP) балансувальники
- Health checks

#### Database Indexing
- Що таке індекс і чому він прискорює SELECT
- Чому індекси сповільнюють INSERT/UPDATE
- Composite index і порядок колонок
- Covering index

#### Основи безпеки
- HTTPS обов'язково
- SQL injection, XSS, CSRF — і як захищатись
- Authentication vs Authorization
- JWT vs session cookies
- OAuth 2.0 — базове розуміння flow

### 3.2 Що маєш вміти на Junior рівні

- Спроєктувати CRUD додаток з БД, API, та фронтом
- Пояснити, як запит доходить від browser до БД
- Написати нормалізовану схему БД для простої задачі (blog, e-commerce)
- Правильно використовувати HTTP методи і status codes
- Налаштувати базовий caching (Redis + cache-aside)

### 3.3 Класичні Junior задачі System Design

- Design a URL Shortener (базова версія)
- Design a Blog Platform
- Design a To-Do List App з синхронізацією
- Design Basic Chat (1-на-1, без масштабування)

### 3.4 Ресурси для Junior

- **Книга:** "Web Scalability for Startup Engineers" — Artur Ejsmont
- **Книга:** "Designing Web APIs" — Brenda Jin (для REST/GraphQL)
- **Сайт:** roadmap.sh/backend — повна карта backend-розвитку
- **Сайт:** developer.mozilla.org — вся мережа/HTTP
- **YouTube:** Hussein Nasser — глибокі пояснення мережі/БД
- **Практика:** побудувати повноцінний pet-project (REST API + БД + фронт)

---

## 4. Middle рівень

**Фокус:** розподілені системи, масштабування, надійність. Ти маєш вміти проєктувати системи на мільйони користувачів.

### 4.1 Теми Middle рівня

#### Масштабування

**Vertical vs Horizontal scaling:**
- Vertical (більше ресурсів на одну машину) — простіше, але має межу
- Horizontal (більше машин) — складніше, але необмежено
- Stateless сервіси масштабуються горизонтально легко; stateful — важко

**Read replicas:**
- Master-slave реплікація
- Читання з replicas, запис на master
- Replication lag і як з ним жити

**Sharding (partitioning):**
- Range-based (за датою, за ID)
- Hash-based (ID % N)
- Directory-based (lookup table)
- Проблеми: rebalancing, hot shards, cross-shard queries

**Consistent Hashing:**
- Як працює коло і віртуальні ноди
- Мінімізує rehashing при додаванні/видаленні нод
- Використовується в Cassandra, DynamoDB, Riak, CDN

#### Теорія розподілених систем

**CAP теорема:**
- Consistency / Availability / Partition tolerance — можеш мати тільки 2 з 3
- У реальному розподіленому світі P обов'язковий → вибір між CP і AP

**PACELC:**
- Розширення CAP: у разі Partition вибір P vs A, Else вибір Latency vs Consistency
- Реалістичніший фреймворк

**ACID vs BASE:**
- ACID — традиційні SQL (строга консистентність)
- BASE — Basically Available, Soft state, Eventual consistency (NoSQL)

**Типи консистентності:**
- Strong consistency — всі бачать однакові дані одразу
- Eventual consistency — зрештою всі синхронізуються
- Read-your-writes — користувач бачить свої зміни
- Monotonic reads — не можна "прочитати назад у часі"
- Causal consistency — зберігається причинно-наслідковий зв'язок

#### Кешування (глибоко)

**Стратегії кешу:**
- **Cache-aside (lazy loading):** додаток сам читає/пише кеш. Найпоширеніше.
- **Read-through:** бібліотека/proxy робить lookup у БД при miss
- **Write-through:** запис одночасно в кеш і БД
- **Write-back (write-behind):** запис у кеш, БД оновлюється async
- **Write-around:** запис тільки в БД, кеш оновиться при читанні

**Eviction policies:**
- LRU, LFU, FIFO, TTL, ARC (adaptive)
- Redis має кілька варіантів

**Проблеми кешу:**
- **Cache stampede** — коли кеш протух, усі одночасно б'ють БД. Рішення: jitter, locks, probabilistic expiry
- **Cache penetration** — запити до неіснуючих ключів. Рішення: Bloom filter, negative caching
- **Cache avalanche** — багато ключів протухає одночасно
- **Hot keys** — один ключ вдаряють мільйони разів

**Рівні кешування:**
- Browser cache
- CDN
- Reverse proxy (nginx, Varnish)
- Application cache (in-memory)
- Distributed cache (Redis, Memcached)
- Database query cache

#### Черги повідомлень (Message Queues)

**Навіщо:**
- Decouple producers and consumers
- Smooth out traffic spikes
- Async processing
- Reliability (persist і повторити)

**Порівняння:**
| Система | Тип | Сильна сторона |
|---------|-----|----------------|
| **RabbitMQ** | Traditional queue | Гнучкий routing, складна логіка |
| **Kafka** | Distributed log | Висока пропускна здатність, replay, streaming |
| **Amazon SQS** | Managed queue | Простий і масштабований |
| **Redis Streams** | Легкий log | Швидкий, простий |
| **NATS** | Pub/sub | Низька затримка |
| **Azure Service Bus** | Enterprise queue | .NET-інтеграція, транзакції |

**Паттерни:**
- Work queue (load distribution)
- Pub/Sub (broadcast)
- Request/Reply (async RPC)
- Dead Letter Queue (DLQ) для "отруєних" повідомлень
- Outbox pattern (гарантована доставка з БД)

**Гарантії доставки:**
- At-most-once (можна втратити)
- At-least-once (можна дублювати — треба idempotency)
- Exactly-once (важко, часто ілюзія)

#### Reliability patterns

- **Retry with exponential backoff + jitter** — не DDoS-ити власний сервіс
- **Circuit Breaker** — перестати бити хворий сервіс, дати йому відновитися (Polly в .NET)
- **Bulkhead** — ізолювати пули ресурсів, щоб провал одного не потягнув інших
- **Timeout** — обов'язково для кожного зовнішнього виклику
- **Fallback** — що повернути, якщо залежність лежить
- **Idempotency** — повторний виклик не шкодить (idempotency key в API)
- **Deduplication** — виявляти і відкидати дублікати

#### Rate Limiting

**Алгоритми:**
- **Token bucket** — найуніверсальніший
- **Leaky bucket** — згладжує burst
- **Fixed window counter** — простий, але "подвоює" на межі вікна
- **Sliding window log** — точний, але дорогий по пам'яті
- **Sliding window counter** — баланс точності і вартості

**Де реалізовувати:** API Gateway, reverse proxy (nginx, Envoy), всередині сервісу (розподілений через Redis).

#### Microservices — коли і як

**Переваги:**
- Незалежний deploy
- Незалежні технології
- Ізоляція збоїв

**Витрати:**
- Мережеві виклики замість function calls
- Distributed transactions складні
- Debug і observability складніші
- Operational overhead

**Коли НЕ варто:** <10 розробників, unclear domain boundaries, monolith ще справляється.

**Концепти:**
- **API Gateway** — один вхід для клієнтів (Kong, Ocelot, AWS API Gateway)
- **Service Discovery** — як сервіси знаходять один одного (Consul, Eureka, Kubernetes DNS)
- **Service Mesh** — sidecar proxy (Istio, Linkerd, Dapr)
- **BFF (Backend-For-Frontend)** — окремий API для кожного типу клієнта

#### Database Design (глибоко)

- Транзакції і рівні ізоляції: як кожен рівень вирішує Dirty Read, Non-Repeatable Read, Phantom Read
- MVCC (Multi-Version Concurrency Control)
- Пессимістичні vs оптимістичні локи
- Connection pooling — обов'язково в продакшені
- Query optimization (EXPLAIN ANALYZE, індекси, partitioning)

#### Observability

**Three pillars:**
- **Metrics** — числові часові ряди (Prometheus + Grafana)
- **Logs** — структуровані текстові записи (ELK stack, Loki, Seq)
- **Traces** — розподілений трейс запиту (Jaeger, Zipkin, OpenTelemetry)

**Golden signals (Google SRE):** Latency, Traffic, Errors, Saturation.

### 4.2 Що маєш вміти на Middle рівні

- Спроєктувати систему на 10M користувачів
- Оцінити потрібну кількість серверів, БД, пропускну здатність
- Вибрати правильну БД під задачу, обґрунтувати
- Розрахувати, де вузьке місце і як його розширити
- Проєктувати API з врахуванням backward compatibility
- Використовувати message queues для decoupling
- Проєктувати надійну систему з retry, timeout, circuit breaker

### 4.3 Класичні Middle задачі

- Design Twitter / Feed системи
- Design Instagram
- Design URL Shortener (масштабовано)
- Design Rate Limiter
- Design Distributed Cache
- Design Notification System
- Design Chat (масштабовано)
- Design Search Autocomplete
- Design News Feed

### 4.4 Ресурси для Middle

- **Книга (must-read):** "Designing Data-Intensive Applications" — Martin Kleppmann. Це **THE book**. Вся сучасна розподілена архітектура в одній книзі.
- **Книга:** "System Design Interview — Volume 1" — Alex Xu. Практичний підхід до класичних задач.
- **Книга:** "Release It!" — Michael Nygard. Reliability patterns.
- **Книга:** "Building Microservices" — Sam Newman.
- **Сайт:** github.com/donnemartin/system-design-primer — величезний free resource.
- **Сайт:** highscalability.com — real-world кейси.
- **Блог:** martin.kleppmann.com — розподілені системи глибоко.
- **Блог:** brooker.co.za/blog — Marc Brooker (AWS).
- **YouTube:** ByteByteGo — короткі візуальні пояснення.
- **YouTube:** Arpit Bhayani — глибокі inside-out пояснення.

---

## 5. Senior рівень

**Фокус:** складні trade-offs, глибоке розуміння внутрішньої кухні, архітектурні рішення на роки вперед.

### 5.1 Теми Senior рівня

#### Consensus і координація

**Навіщо:** розподілені ноди мають домовитись про спільне рішення (хто лідер, який порядок операцій).

- **Two-phase commit (2PC)** — гарантії, але блокується при збоях
- **Three-phase commit (3PC)** — спроба виправити, не ідеально
- **Paxos** — класичний, але складний
- **Raft** — простіший і популярніший (etcd, Consul, CockroachDB)
- **ZAB** — протокол ZooKeeper
- **Viewstamped Replication**

**Для твоєї тези:** Raft — must-know, оскільки він фактично стандарт для сучасних систем.

**Leader election:** як обрати одну ноду координатором (bully, Raft-based).

#### Розподілені транзакції

- **Saga pattern** — компенсуючі транзакції замість 2PC
  - Choreography (подієвий) vs Orchestration (координатор)
- **Outbox pattern** — гарантована публікація подій разом з БД-транзакцією
- **CDC (Change Data Capture)** — Debezium, Kafka Connect
- **TCC (Try-Confirm-Cancel)** — альтернатива Saga

#### Event-driven architecture

- **Event Sourcing** — зберігаємо події, не стан. Стан — це згортка подій.
- **CQRS** — розділення моделей для читання і запису
- **Event Streaming** — Kafka як source of truth
- **Event-carried state transfer** vs event notification vs event-sourced

**Переваги:** аудит, replay, temporal queries.
**Недоліки:** складність, eventual consistency, schema evolution.

#### Storage deep dive

**Database internals:**
- **B-tree** — як працюють сторінки, splits, merges (PostgreSQL, MySQL InnoDB)
- **LSM-tree** — memtable + sorted strings (Cassandra, RocksDB, LevelDB)
- **WAL (Write-Ahead Log)** — чому це must-have для durability
- **Bloom filters** всередині LSM для прискорення read
- **Compaction** — чому і як працює

**Storage engines:**
- InnoDB (MySQL default)
- Postgres heap + btree
- RocksDB (Facebook LSM)
- FoundationDB

**Backup & Recovery:**
- Full / Incremental / Differential backups
- Point-in-time recovery через WAL
- Cross-region replication
- RPO (Recovery Point Objective), RTO (Recovery Time Objective)

#### Stream Processing

- **Apache Kafka Streams** — stream processing з Kafka
- **Apache Flink** — low-latency stateful streaming
- **Apache Spark Streaming** — mini-batch
- Windowing (tumbling, sliding, session)
- Exactly-once processing
- Watermarks, late events
- Joins streams з таблицями

#### Data Architecture

- **Data Lake** — сирі дані в будь-якому форматі (S3, HDFS)
- **Data Warehouse** — структуровані, аналітичні (Snowflake, Redshift, BigQuery)
- **Lakehouse** — гібрид (Databricks, Iceberg)
- **OLTP vs OLAP** — різна оптимізація
- **ETL vs ELT** — коли трансформувати
- **Star schema, Snowflake schema** — для аналітики

#### Multi-region та Geo-distribution

- Active-Passive vs Active-Active
- Global load balancing (DNS-based, anycast)
- Data residency / GDPR обмеження
- Cross-region replication та його вартість/затримка
- Conflict resolution (CRDTs, Last-Write-Wins, vector clocks)
- Edge computing (Cloudflare Workers, Fastly, AWS Lambda@Edge)

#### Capacity Planning

- Forecasting (лінійний зростання, сезонність)
- Headroom (зазвичай 30-50%)
- Auto-scaling vs pre-provisioning
- Cost vs reliability trade-off

#### Reliability Engineering (SRE)

- **SLA / SLO / SLI** — різниця і використання
- Error budgets
- Postmortems (blameless)
- Chaos Engineering (Netflix Chaos Monkey, Gremlin)
- Canary deployments, blue-green, rolling
- Feature flags (LaunchDarkly, ConfigCat)

#### Security Architecture

- **Zero Trust** — ніколи не довіряй, завжди перевіряй
- **mTLS** — взаємна автентифікація сервіс-до-сервісу
- **Secrets management** (HashiCorp Vault, AWS Secrets Manager, Azure Key Vault)
- **OAuth 2.0 + OIDC** глибоко (всі flows: Authorization Code, PKCE, Client Credentials, Device Code)
- **RBAC vs ABAC** — різні моделі доступу
- **API security** — rate limiting, input validation, HMAC signatures
- **Encryption at rest / in transit** — коли що обов'язкове

#### Performance engineering

- Profiling (flamegraphs, CPU/memory profilers)
- Понимання GC impact (в .NET: gen0/1/2, LOH, Server GC)
- Зменшення allocations (Span<T>, ArrayPool, object pooling)
- Async hot paths
- Connection pooling (DB, HTTP)
- Batching vs streaming

### 5.2 Що маєш вміти на Senior рівні

- Проєктувати системи на 100M+ користувачів
- Обговорювати trade-offs на глибокому рівні (CAP, consistency models)
- Пояснити внутрішню роботу БД, черг, кешу
- Розуміти, коли і як використовувати event sourcing / CQRS
- Проєктувати multi-region системи
- Впливати на архітектурні рішення команди
- Проводити postmortems
- Обирати між alternatives з обґрунтуванням

### 5.3 Класичні Senior задачі

- Design Uber / Lyft
- Design YouTube / Netflix streaming
- Design Google Drive / Dropbox
- Design Payment System (Stripe-like)
- Design Distributed File System
- Design Search Engine (Google/Elasticsearch)
- Design Recommendation System
- Design Distributed Scheduler (cron at scale)
- Design a Stock Exchange
- Design Ads System (Google Ads-like)
- Design Real-time Bidding
- Design Collaborative Editor (Google Docs)

### 5.4 Ресурси для Senior

- **Книга:** "Database Internals" — Alex Petrov. Як насправді працюють БД всередині.
- **Книга:** "Designing Distributed Systems" — Brendan Burns (автор Kubernetes).
- **Книга:** "Enterprise Integration Patterns" — Hohpe & Woolf. Класика паттернів інтеграції.
- **Книга:** "Site Reliability Engineering" — Google, безкоштовно онлайн.
- **Книга:** "Fundamentals of Software Architecture" — Mark Richards, Neal Ford.
- **Книга:** "Building Event-Driven Microservices" — Adam Bellemare.
- **Блог:** aphyr.com (Kyle Kingsbury / Jepsen) — тестування розподілених БД.
- **Блог:** cidrdb.org — papers про БД research.
- **Papers:** "Dynamo", "BigTable", "Spanner", "Kafka", "Chubby", "Paxos made Simple", "Raft" — класичні papers, обов'язково прочитати.

#### Папери, які варто знати

| Папір | Про що |
|-------|--------|
| **Paxos Made Simple** (Lamport) | Класичний consensus |
| **In Search of an Understandable Consensus Algorithm** (Ongaro, Ousterhout) | Raft |
| **Dynamo: Amazon's Highly Available Key-value Store** | Фундамент DynamoDB, Cassandra |
| **Bigtable** (Google) | Фундамент HBase, Cassandra |
| **The Google File System** | Фундамент HDFS |
| **MapReduce** (Google) | Фундамент Hadoop |
| **Spanner** (Google) | Globally distributed SQL |
| **Kafka: a Distributed Messaging System** | Log-based streaming |
| **CAP Twelve Years Later** (Brewer) | Переосмислення CAP |
| **Time, Clocks, and the Ordering of Events** (Lamport) | Логічні години |
| **Aurora** (Amazon) | Storage-compute separation |

---

## 6. Staff/Principal рівень

**Фокус:** не "як побудувати", а "як організувати побудову" + глибокий технічний вплив.

### 6.1 Технічні теми

- **Платформи** — побудувати платформу, на якій інші команди будують
- **Multi-tenant системи** з ізоляцією
- **Billing і metering** масштабовано (як Stripe, AWS)
- **Compliance-aware architecture** (SOC2, ISO 27001, HIPAA, PCI-DSS)
- **ML Systems at scale** — Feature store, model serving, A/B тести
- **Quantum-safe cryptography** — підготовка до post-quantum ери
- **Edge / IoT architecture**

### 6.2 Не-технічні теми (ключові для Staff+)

- **Technical strategy** — оцінка, куди інвестувати інженерні ресурси
- **Cross-team alignment** — узгодження архітектури між командами
- **Written communication** — технічні RFC, design docs, tech radar
- **Mentorship** — розвивати senior-інженерів
- **Operational excellence** — incident command, reliability review
- **Vendor selection** — коли купити, коли побудувати, коли open source

### 6.3 Ресурси

- **Книга:** "The Staff Engineer's Path" — Tanya Reilly.
- **Книга:** "Staff Engineer: Leadership beyond the management track" — Will Larson.
- **Книга:** "An Elegant Puzzle" — Will Larson (engineering management).
- **Блог:** lethain.com (Will Larson).
- **Блог:** staffeng.com — інтерв'ю зі staff-інженерами FAANG.

---

## 7. Класичні задачі System Design

Список задач, впорядкований за складністю. Кожну варто розібрати — як мінімум прочитати чужий розбір, як максимум — спроєктувати самостійно.

### 7.1 Розминка (Junior-Middle)

- **URL Shortener** (bit.ly) — hashing, collision, кешування, БД
- **Pastebin** — storage, expiration, privacy
- **To-do list** з синхронізацією — offline-first, conflict resolution

### 7.2 Класика Middle

- **Twitter / News Feed** — fan-out on write vs on read
- **Instagram** — image storage, newsfeed, social graph
- **Rate Limiter** — алгоритми, distributed rate limiting
- **Distributed Cache** — consistent hashing, replication
- **Notification System** — push, email, SMS, priority
- **Chat (WhatsApp / Slack)** — message ordering, delivery guarantees, online presence
- **Autocomplete / Typeahead** — Trie, ranking, freshness
- **Leaderboard** — Redis sorted sets, top-K, real-time updates
- **Web Crawler** — BFS, politeness, dedup, scale
- **Log aggregation (ELK-like)** — ingestion, storage, search

### 7.3 Складні Senior задачі

- **Uber / Lyft** — geo-indexing (quadtree, S2, geohash), matching, ETA
- **YouTube / Netflix** — video encoding, CDN, recommendation
- **Dropbox / Google Drive** — file sync, deduplication, chunking
- **Facebook Messenger** — message delivery at scale, E2E encryption
- **Search Engine (mini-Google)** — crawling, inverted index, ranking (PageRank, BM25)
- **Payment System (Stripe-like)** — idempotency, double-entry bookkeeping, reconciliation
- **Ads System** — targeting, real-time bidding, budgeting
- **Ride-sharing matching** — supply/demand, surge pricing
- **Flash sale / Ticket booking** — inventory, concurrency, fairness
- **Real-time collaborative editor (Google Docs)** — OT or CRDTs
- **Video call (Zoom)** — WebRTC, SFU vs MCU, scalability
- **Distributed job scheduler (cron at scale)** — Quartz, Temporal
- **Metrics system (Prometheus-like)** — time-series DB, downsampling
- **Distributed File System (HDFS-like)** — NameNode, chunks, replication

### 7.4 Hyper-scale

- **Design Kafka** — partition, replication, log-structured storage
- **Design DynamoDB** — partitioning, Paxos/replication, global tables
- **Design S3** — eventual consistency, versioning, multi-region
- **Design BigQuery** — columnar storage, distributed query engine
- **Design Spanner** — TrueTime, synchronous global replication
- **Design a blockchain** — consensus, merkle trees, smart contracts

---

## 8. Рамка відповіді на інтерв'ю

На інтерв'ю в тебе зазвичай 45-60 хв на одну задачу. Дотримуйся структури.

### 8.1 Step 1: Requirements clarification (5-7 хв)

**Functional requirements — уточни обсяг:**
- Хто користувачі?
- Які ключові фічі?
- Що точно не в scope?

**Non-functional requirements:**
- Scale: скільки користувачів, QPS, обсяг даних
- Latency target (p50, p99)
- Availability (скільки дев'яток)
- Consistency (strong/eventual)
- Durability

**Приклад:** "Before I start, кілька питань. Ми проєктуємо Twitter. Фокус на core-фічі: posting tweets, timeline, follow? Ok, 200M DAU, середньо 100M tweets на день, 10:1 read/write ratio. Latency goal для timeline — p99 <200ms. Eventual consistency прийнятна для timeline."

### 8.2 Step 2: Scale estimation (3-5 хв)

Числа на дошці:
- QPS (write і read окремо)
- Обсяг даних на рік (5 років)
- Bandwidth (in / out)
- Пам'ять потрібна для кешу

**Приклад:**
- 100M tweets/day = ~1150 writes/sec, peak ~3000
- Read QPS = 30 000 (при 10:1)
- Tweet size ~300B → 100M × 300B = 30 GB/day → 10 TB/рік

### 8.3 Step 3: API Design (3-5 хв)

Ключові ендпоінти:
```
POST /v1/tweets
  Body: { content, mediaUrls? }
  Returns: { tweetId, createdAt }

GET /v1/users/{id}/timeline?cursor=X&limit=20
  Returns: { tweets: [...], nextCursor }

POST /v1/users/{id}/follow
```

Для gRPC чи GraphQL — пояси, чому.

### 8.4 Step 4: High-Level Architecture (5-10 хв)

Малюй "коробки і стрілки":
- Клієнти (mobile, web)
- Load balancer
- API Gateway
- Сервіси (Tweet Service, Timeline Service, User Service)
- БД (primary + replicas)
- Кеш (Redis)
- Queue (Kafka)
- CDN / Object storage (для медіа)

### 8.5 Step 5: Data Model (5 хв)

- Основні таблиці/колекції
- Ключові поля і типи
- Індекси
- Яка БД (чому SQL, чому NoSQL)

**Приклад:**
```
tweets (Cassandra — write-heavy):
  PK: (user_id, tweet_id DESC)
  fields: content, media_urls, created_at

followers (Cassandra):
  PK: (user_id, follower_id)

users (PostgreSQL — strong consistency):
  id, username, email, created_at
```

### 8.6 Step 6: Deep Dive (10-15 хв)

Інтерв'юер спитає "а як працюватиме X?". Вибери 1-2 критичні компоненти і розглянь детально:
- Алгоритми
- Trade-offs
- Failure modes
- Скалювання

**Для Twitter:** fan-out стратегія для timeline — fan-out on write (Redis timelines для кожного юзера) для обычних юзерів, on-read для celebrities (10M+ followers). Hybrid approach.

### 8.7 Step 7: Bottlenecks & Trade-offs (5 хв)

- Де першим зламається? (hot shard, DB write, cache stampede)
- Як моніторити? (метрики, алерти)
- Що покращити в наступній ітерації?
- Якщо ми зробимо Z замість Y — які trade-offs?

### 8.8 Антипаттерни на інтерв'ю

- Прямо кинутись малювати, не запитавши про requirements
- Одразу кидатися мікросервісами, Kubernetes, Kafka без обґрунтування
- Говорити тільки загальне, не показувати глибини
- Ігнорувати failure cases
- Не робити оцінки числом

---

## 9. Ресурси

### 9.1 Книги (впорядковано за пріоритетом)

**Must-read:**
1. **"Designing Data-Intensive Applications"** — Martin Kleppmann. Єдина книга, яку всі погоджуються рекомендувати.
2. **"System Design Interview Vol 1 & 2"** — Alex Xu. Практично, з картинками, під інтерв'ю.
3. **"Release It!" 2nd ed.** — Michael Nygard. Reliability patterns.

**Сильно рекомендую:**
4. **"Building Microservices"** — Sam Newman.
5. **"Database Internals"** — Alex Petrov.
6. **"Site Reliability Engineering"** — Google (безкоштовно: sre.google/books).
7. **"The Site Reliability Workbook"** — Google.

**Глибше/спеціалізовано:**
8. **"Fundamentals of Software Architecture"** — Richards, Ford.
9. **"Software Architecture: The Hard Parts"** — Richards, Ford. Про складні trade-offs у distributed architectures.
10. **"Enterprise Integration Patterns"** — Hohpe, Woolf.
11. **"Domain-Driven Design"** — Eric Evans. Якщо проєктуєш бізнес-системи.
12. **"Patterns of Enterprise Application Architecture"** — Martin Fowler.
13. **"Building Event-Driven Microservices"** — Adam Bellemare.
14. **"Clean Architecture"** — Robert Martin.
15. **"The Phoenix Project"** + **"The Unicorn Project"** — fiction про DevOps.

### 9.2 Онлайн-ресурси

**GitHub repos:**
- github.com/donnemartin/system-design-primer — найвідоміший free-resource
- github.com/ByteByteGoHq/system-design-101 — візуальний
- github.com/checkcheckzz/system-design-interview — collection задач
- github.com/karanpratapsingh/system-design — структурований конспект

**Сайти:**
- **highscalability.com** — case studies реальних систем
- **martin.kleppmann.com/papers** — папери і статті Kleppmann
- **brooker.co.za/blog** — Marc Brooker з AWS, глибока технічна аналітика
- **aws.amazon.com/builders-library** — як AWS проєктує свої системи
- **netflixtechblog.com** — інженерний блог Netflix
- **engineering.fb.com** — Meta engineering
- **eng.uber.com** — Uber engineering
- **discord.com/blog/engineering-infrastructure** — Discord
- **bytebytego.com** — Alex Xu's blog і newsletter

### 9.3 YouTube-канали

- **ByteByteGo** (Alex Xu) — короткі візуальні пояснення
- **Hussein Nasser** — глибокі пояснення мережі, БД, fundamentals
- **Arpit Bhayani** — асинхронні розповіді про inside внутрішньої роботи систем
- **Gaurav Sen** — класичні system design задачі
- **Jordan has no life** — системний дизайн інтерв'ю пояснення
- **Tech Dummies Narendra L** — Netflix engineering
- **System Design Interview** — specifically для інтерв'ю
- **InfoQ** — технічні доповіді з конференцій
- **GOTO Conferences** — архітектурні доповіді

### 9.4 Платні курси

- **Grokking the System Design Interview** (educative.io) — класика для prep
- **Grokking the Advanced System Design** (educative.io) — для senior
- **Designing Data-Intensive Applications** (O'Reilly) — курс на базі книги
- **ByteByteGo** — підписка на курси Алекса Сю
- **Exponent** (tryexponent.com) — mock interviews
- **Educative.io** — багато system design треків

### 9.5 Інженерні блоги (для case studies)

Вивчати реальні кейси — найкращий спосіб зрозуміти, як рішення приймаються в продакшені.

- Netflix Tech Blog
- Uber Engineering
- Airbnb Engineering
- Stripe Engineering
- Shopify Engineering
- Discord Engineering
- Slack Engineering
- Cloudflare Blog
- Twitter Engineering
- GitHub Engineering
- LinkedIn Engineering
- Meta (Facebook) Engineering
- Amazon Builders' Library
- Google Research
- Microsoft Research

### 9.6 Конференції (записи на YouTube)

- **QCon** — software architecture
- **Strange Loop** — distributed systems, programming
- **GOTO** — architecture
- **re:Invent** (AWS) — специфічно AWS
- **Kafka Summit**
- **KubeCon** — Kubernetes і cloud native

---

## 10. Таймлайн вивчення

Припускаючи ~10 годин на тиждень:

### Roadmap для Middle-рівня (6-9 місяців)

| Тиждень | Тема |
|---------|------|
| 1-2 | Networking fundamentals, HTTP/TCP deep dive |
| 3-4 | OS basics, async I/O, threads |
| 5-6 | Databases basics, ACID, індекси, transactions |
| 7-8 | Book: DDIA ch. 1-4 (foundations, encoding, data models) |
| 9-10 | Caching (cache-aside, Redis internals, eviction) |
| 11-12 | DDIA ch. 5-6 (replication, partitioning) |
| 13-14 | Message queues, Kafka basics |
| 15-16 | DDIA ch. 7-9 (transactions, distributed problems, consistency) |
| 17-18 | Load balancing, consistent hashing, CDN |
| 19-20 | Microservices, API Gateway, service discovery |
| 21-22 | Reliability patterns (Polly, circuit breaker, retry, timeout) |
| 23-24 | Observability (Prometheus, Grafana, OpenTelemetry) |
| 25-28 | Practice: 10 класичних задач по 2-3 години кожна |
| 29-32 | Mock interviews |
| 33-36 | Refresh слабких тем |

### Roadmap для Senior (ще +6 місяців після Middle)

| Тиждень | Тема |
|---------|------|
| 1-2 | DDIA ch. 10-12 (batch, stream processing, future) |
| 3-4 | Consensus: Paxos і Raft глибоко, прочитати папери |
| 5-6 | Event Sourcing, CQRS, Saga |
| 7-8 | Database internals (LSM-tree, B-tree, WAL) — Database Internals book |
| 9-10 | Multi-region, geo-distribution, CRDTs |
| 11-12 | Security architecture, Zero Trust, mTLS, secrets |
| 13-16 | Практика: складні задачі (Uber, Payments, Drive, Search) |
| 17-20 | Читання real-world case studies (High Scalability, engineering blogs) |
| 21-24 | Папери: Dynamo, Spanner, BigTable, Kafka, Aurora |

### Сигнали готовності

**Готовий до Middle-рівня senior-співбесіди:**
- Можеш за 45 хв спроєктувати Twitter чи Instagram без допомоги
- Оціниш scale у числах
- Обираєш БД з обґрунтуванням
- Розумієш CAP і пояснюєш це без формул

**Готовий до Senior-рівня:**
- Розумієш Raft на рівні "можу пояснити algorithm on whiteboard"
- Знаєш, чим LSM-tree відрізняється від B-tree і коли що
- Робив deep-dive у 3+ engineering blog posts і можеш пояснити рішення
- Проєктував системи в production з consistency/availability trade-offs
- Вмієш сказати "я б обрав X, але варіант Y теж валідний, ось trade-off"

---

## 11. .NET-специфіка

Оскільки твій основний стек — C#/.NET, ось як теми System Design мапляться на .NET екосистему.

### 11.1 Фреймворки/бібліотеки

**Веб:**
- **ASP.NET Core** — основа backend
- **Minimal APIs** (від .NET 6) — швидкий старт
- **gRPC** — для внутрішніх сервісів, .NET має першокласну підтримку
- **SignalR** — real-time (WebSockets, Server-Sent Events)
- **HotChocolate** — GraphQL (ти вже використовував)
- **YARP** — reverse proxy/API gateway від Microsoft

**Reliability:**
- **Polly** — retry, circuit breaker, timeout, bulkhead. Стандарт в .NET.
- **HttpClientFactory** — правильне управління HttpClient
- **.NET Resilience** (Microsoft.Extensions.Resilience) — нова обгортка над Polly

**Кешування:**
- **IMemoryCache** — in-process cache
- **IDistributedCache** — абстракція над Redis/Memcached/SQL Server
- **StackExchange.Redis** — Redis клієнт #1 для .NET
- **FusionCache** — продвинута бібліотека з anti-stampede

**Messaging:**
- **MassTransit** — абстракція над RabbitMQ/Azure Service Bus/Amazon SQS/Kafka
- **NServiceBus** — комерційний enterprise-messaging
- **Confluent.Kafka** — офіційний Kafka клієнт для .NET
- **Azure Service Bus SDK** — якщо Azure
- **MediatR** — in-process messaging (але не завжди потрібен)

**Event Sourcing / CQRS:**
- **Marten** — Postgres як event store + document DB
- **EventStoreDB** — спеціалізована БД для events
- **Wolverine** — messaging і CQRS framework від автора Marten

**Distributed / Actor model:**
- **Orleans** — virtual actors від Microsoft. Відмінний для stateful distributed systems. **Тобі цікаво для тези.**
- **Akka.NET** — порт Akka з JVM
- **Proto.Actor** — легкий actor model
- **Dapr** — building blocks для distributed apps (service invocation, state, pub/sub)

**Observability:**
- **OpenTelemetry.NET** — стандарт для metrics/traces/logs
- **Serilog** — структуроване логування
- **Prometheus-net** — Prometheus metrics
- **Application Insights** — якщо Azure

**Data access:**
- **Entity Framework Core** — ORM
- **Dapper** — micro-ORM, швидкий
- **Npgsql** — Postgres driver (найкращий)
- **MongoDB.Driver** — MongoDB

### 11.2 Важливі .NET-specific теми

**Performance:**
- Розуміти GC (Workstation vs Server GC, Background GC)
- `Span<T>`, `Memory<T>`, `ArrayPool<T>` — zero-allocation hot paths
- `IAsyncEnumerable<T>` — streaming
- ValueTask — для оптимізації sync completion
- `ConfigureAwait(false)` — коли потрібен
- Native AOT (.NET 8+) — для startup-heavy workloads і containers

**Hosting:**
- **Kestrel** як web server
- **HTTP/2, HTTP/3** підтримка
- **YARP** як reverse proxy
- **Minimal hosting model**
- **BackgroundService** для довготривалих задач
- **IHostedService**

**Deployment:**
- Docker з multi-stage build
- Kubernetes з .NET — health checks, readiness probes
- Azure App Service, Container Apps, AKS
- AWS Elastic Beanstalk, ECS, EKS

### 11.3 Архітектурні підходи в .NET

- **Clean Architecture** — Jason Taylor template, Ardalis template
- **Vertical Slice Architecture** — Jimmy Bogard (часто краще ніж Clean)
- **Onion Architecture**
- **DDD з MediatR** — CQRS в простому вигляді

### 11.4 Ресурси specifically для .NET

- **YouTube:** Nick Chapsas — сучасні практики
- **YouTube:** Milan Jovanović — архітектура в .NET
- **YouTube:** Tim Corey — фундаменти
- **YouTube:** Elton Stoneman — Docker/Kubernetes з .NET
- **Book:** "Dependency Injection Principles, Practices, and Patterns" — Mark Seemann
- **Book:** "Pro ASP.NET Core" — Adam Freeman
- **Blog:** jimmybogard.com — MediatR, архітектура
- **Blog:** andrewlock.net — глибокі технічні пости

### 11.5 Використання для тези

Для децентралізованої обчислювальної системи з кастомним протоколом:

- **Orleans** — природний вибір для distributed state (Master/Worker як grains)
- **Channels** (System.Threading.Channels) — внутрішній messaging
- **Span<T>** — zero-copy для протоколу над TCP
- **SslStream / кастомний TLS** через OpenSSL via P/Invoke
- **Polly** — retry/circuit breaker для комунікації Master ↔ Worker
- **OpenTelemetry** — розподілене трасування через ноди
- **Raft** — якщо потрібна координація Masters (etcd-like або dotnet-raft бібліотека)

---

## Фінальна порада

System Design — це **не те, що вчиться за тиждень перед інтерв'ю**. Це накопичувальна дисципліна:
1. **Читай реальні інженерні блоги** — 1-2 статті на тиждень. За рік у голові буде "бібліотека" рішень.
2. **Проєктуй на роботі мислячи про scale** — навіть якщо поки не треба.
3. **Дебажучи інциденти — думай системно.** Чому саме тут зламалось? Яка була схована залежність?
4. **Роби постмортеми** — навіть для власних помилок.
5. **Говори з іншими інженерами** — 30 хвилин whiteboard з досвідченим колегою вартують 10 годин самостійного читання.

Найбільша пастка — намагатися запам'ятати архітектури класичних систем. Це даремно. Краще зрозуміти **принципи** і вміти їх застосовувати до нових задач.
