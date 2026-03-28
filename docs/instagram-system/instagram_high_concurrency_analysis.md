# Phân Tích Luồng Xử Lý Đồng Thời Cao - Instagram

> Instagram xử lý **hàng triệu requests/giây** từ 2+ tỷ users. Tài liệu này phân tích chi tiết các kỹ thuật xử lý high concurrency.

---

## 1. Vấn Đề "Celebrity Problem" - Hybrid Fan-out

Khi 1 celebrity có 100M followers đăng ảnh, hệ thống KHÔNG THỂ push tới 100M feed cùng lúc.

### Giải pháp: Hybrid Push/Pull

```mermaid
flowchart TB
    subgraph "User đăng bài"
        POST["📸 New Post Created"]
    end

    POST --> CHECK{"👤 User type?"}

    CHECK -->|"Normal User<br/>(< 500K followers)"| PUSH["📤 Fan-out on WRITE<br/>(Push Model)"]
    CHECK -->|"Celebrity<br/>(> 500K followers)"| PULL["📥 Fan-out on READ<br/>(Pull Model)"]

    subgraph "Push Path - Async"
        PUSH --> KAFKA1["Kafka: post.created"]
        KAFKA1 --> WORKERS["Celery Workers<br/>(batch 1000 followers/task)"]
        WORKERS --> REDIS_FEED["Redis Sorted Set<br/>follower_feed:{user_id}"]
    end

    subgraph "Pull Path - On Demand"
        PULL --> CELEB_CACHE["Cache: celebrity_posts:{user_id}<br/>Chỉ lưu posts gần nhất"]
    end

    subgraph "Khi User mở Feed"
        OPEN["📱 User opens app"] --> MERGE
        REDIS_FEED --> MERGE["🔀 Merge & Rank"]
        CELEB_CACHE --> MERGE
        MERGE --> ML["🤖 ML Ranking Model"]
        ML --> RESPONSE["📰 Final Feed"]
    end

    style PUSH fill:#51cf66,color:#fff
    style PULL fill:#ffd43b,color:#333
    style MERGE fill:#4c6ef5,color:#fff
```

| Aspect | Push (Normal User) | Pull (Celebrity) |
|---|---|---|
| **Trigger** | Khi user đăng bài | Khi follower mở feed |
| **Write load** | O(N followers) - async | O(1) - chỉ cache post |
| **Read latency** | Rất thấp (feed sẵn sàng) | Cao hơn (phải merge) |
| **Áp dụng** | < 500K followers | > 500K followers |

---

## 2. Thundering Herd - Cache Stampede

Khi cache key phổ biến hết hạn → hàng triệu requests cùng đổ vào DB.

### Giải pháp: Promise Caching + Request Coalescing

```mermaid
sequenceDiagram
    participant R1 as Request 1
    participant R2 as Request 2
    participant R3 as Request 3
    participant Cache as Memcached
    participant DB as PostgreSQL

    Note over R1,DB: Scenario: Cache key "user:123:profile" expired

    R1->>Cache: GET user:123:profile
    Cache-->>R1: ❌ MISS

    R1->>Cache: SET user:123:profile = PROMISE (lock)
    R1->>DB: SELECT * FROM users WHERE id=123

    R2->>Cache: GET user:123:profile
    Cache-->>R2: ⏳ PROMISE found → WAIT

    R3->>Cache: GET user:123:profile
    Cache-->>R3: ⏳ PROMISE found → WAIT

    DB-->>R1: {name: "alice", ...}
    R1->>Cache: SET user:123:profile = {data} (TTL: 5min)

    Cache-->>R2: ✅ Data ready → return
    Cache-->>R3: ✅ Data ready → return

    Note over R1,DB: Chỉ 1 query tới DB thay vì 1 triệu queries!
```

### 3 kỹ thuật chống Thundering Herd

| Kỹ thuật | Mô tả |
|---|---|
| **Promise Caching** | Request đầu tiên lock cache key, các request sau đợi kết quả |
| **Soft TTL + Background Refresh** | Cache trả stale data, đồng thời refresh async ở background |
| **Lease-based Locking** | Memcached cấp "lease" cho 1 request, các request khác nhận stale value |

---

## 3. Celebrity Like Problem - Sharded Counters

Khi 1 bài post nhận **hàng triệu likes/phút**, update 1 row `like_count` → **lock contention** nghiêm trọng.

```mermaid
flowchart TB
    subgraph "❌ Naive: Single Counter"
        A1["User Like"] --> ROW["UPDATE posts<br/>SET like_count = like_count + 1<br/>WHERE id = 123"]
        A2["User Like"] --> ROW
        A3["User Like"] --> ROW
        ROW --> LOCK["🔒 Row Lock Contention!<br/>1 triệu writes tranh 1 lock"]
    end

    subgraph "✅ Solution: Sharded Counters"
        B1["User Like"] --> KAFKA2["Kafka: like.created"]
        B2["User Like"] --> KAFKA2
        B3["User Like"] --> KAFKA2

        KAFKA2 --> REDIS2["Redis INCR<br/>post:123:likes → instant UI"]

        KAFKA2 --> W1["Worker → Shard 0<br/>post_likes_0: +334"]
        KAFKA2 --> W2["Worker → Shard 1<br/>post_likes_1: +333"]
        KAFKA2 --> W3["Worker → Shard 2<br/>post_likes_2: +333"]

        W1 & W2 & W3 --> AGG["⏰ Periodic Aggregation<br/>SUM(all shards) = total likes"]
        AGG --> PG2["PostgreSQL<br/>Materialized like_count"]
    end

    style LOCK fill:#ff6b6b,color:#fff
    style REDIS2 fill:#51cf66,color:#fff
    style AGG fill:#4c6ef5,color:#fff
```

**Luồng chi tiết:**

1. **User nhấn Like** → API trả lời ngay *202 Accepted*
2. **Redis `INCR`** → counter tăng atomic, UI cập nhật instant
3. **Kafka event** → buffer absorb traffic spikes
4. **Workers** ghi vào N shards song song (không tranh lock)
5. **Periodic job** aggregate tổng shards → ghi vào DB chính

---

## 4. Database Concurrency Control

```mermaid
flowchart TB
    subgraph "Connection Management"
        APP["Django App<br/>(nhiều processes)"]
        POOL["PgBouncer<br/>Connection Pooling<br/>(10K connections → 200 DB connections)"]
        APP --> POOL
    end

    subgraph "Write Path"
        POOL -->|Writes only| LEADER["🟠 PostgreSQL Leader"]
    end

    subgraph "Read Path (90% traffic)"
        POOL -->|Reads| R1["🔵 Replica 1"]
        POOL -->|Reads| R2["🔵 Replica 2"]
        POOL -->|Reads| R3["🔵 Replica 3"]
        POOL -->|Reads| R4["🔵 Replica 4"]
    end

    LEADER -->|Async Replication| R1 & R2 & R3 & R4

    subgraph "Hot Data Protection"
        CACHE_L["Memcached Layer<br/>99% cache hit rate"]
        CACHE_L --> POOL
    end

    style LEADER fill:#ff922b,color:#fff
    style CACHE_L fill:#51cf66,color:#fff
```

| Kỹ thuật | Mục đích | Chi tiết |
|---|---|---|
| **PgBouncer** | Connection pooling | 10K app connections → 200 DB connections |
| **Read Replicas** | Phân tải reads | 90% traffic → replicas, 10% → leader |
| **Optimistic Locking** | Tránh row locks | `WHERE version = N` thay vì `SELECT FOR UPDATE` |
| **Batch Writes** | Giảm round-trips | Gom nhiều writes thành 1 transaction |
| **Cache-aside** | Giảm DB load | 99%+ cache hit rate → DB chỉ nhận 1% traffic |

---

## 5. Async Processing Pipeline

```mermaid
flowchart LR
    subgraph "Critical Path (< 200ms)"
        REQ["📱 API Request"]
        AUTH2["🔐 Auth Check"]
        BIZ["⚡ Core Logic"]
        RESP["📤 Response"]
        REQ --> AUTH2 --> BIZ --> RESP
    end

    BIZ -->|"Fire & Forget"| KAFKA3["📨 Kafka"]

    subgraph "Non-Critical Path (Async)"
        KAFKA3 --> T1["🎬 Video Transcoding<br/>(minutes)"]
        KAFKA3 --> T2["📧 Email Notification<br/>(seconds)"]
        KAFKA3 --> T3["📊 Analytics Update<br/>(seconds)"]
        KAFKA3 --> T4["🔍 Search Index Update<br/>(seconds)"]
        KAFKA3 --> T5["🤖 ML Feature Update<br/>(seconds)"]
    end

    subgraph "Worker Fleet"
        T1 --> WF["Auto-scaling Workers<br/>Scale theo queue depth"]
        T2 --> WF
        T3 --> WF
        T4 --> WF
        T5 --> WF
    end

    style REQ fill:#4c6ef5,color:#fff
    style RESP fill:#51cf66,color:#fff
    style KAFKA3 fill:#ff922b,color:#fff
```

**Nguyên tắc core:** Không bao giờ đặt operation nặng trên critical path.

| Path | SLA | Ví dụ |
|---|---|---|
| **Critical** | < 200ms | Auth, đọc feed, POST like |
| **Near-realtime** | < 5s | Push notification, feed update |
| **Background** | < 5min | Video transcode, email, analytics |

---

## 6. Rate Limiting & Circuit Breaker

```mermaid
flowchart TB
    REQ2["Incoming Request<br/>~1M req/s"] --> RL

    subgraph "Layer 1: Rate Limiting"
        RL{"Rate Limiter<br/>(Token Bucket)"}
        RL -->|"Over limit"| REJECT["429 Too Many Requests"]
        RL -->|"Under limit"| CB
    end

    subgraph "Layer 2: Circuit Breaker"
        CB{"Circuit Breaker<br/>State?"}
        CB -->|"OPEN<br/>(service down)"| FALLBACK["⚠️ Fallback Response<br/>(cached/default)"]
        CB -->|"CLOSED<br/>(healthy)"| SERVICE["✅ Service Call"]
        CB -->|"HALF-OPEN<br/>(testing)"| PROBE["🔍 Probe Request"]
        PROBE -->|"Success"| CLOSE["Close Circuit"]
        PROBE -->|"Fail"| OPEN2["Open Circuit"]
    end

    subgraph "Layer 3: Bulkhead"
        SERVICE --> BH["Thread Pool Isolation<br/>Feed: 200 threads<br/>DM: 100 threads<br/>Search: 50 threads"]
    end

    style REJECT fill:#ff6b6b,color:#fff
    style FALLBACK fill:#ffd43b,color:#333
    style SERVICE fill:#51cf66,color:#fff
```

### Rate Limiting tầng tầng lớp lớp

| Tầng | Loại | Limit |
|---|---|---|
| **CDN/Edge** | IP-based | 1000 req/min per IP |
| **API Gateway** | User-based (JWT) | 200 req/min per user |
| **Service Level** | Endpoint-based | POST /like: 60/min, GET /feed: 30/min |
| **Database** | Connection-based | Max 200 connections via PgBouncer |

---

## 7. Tổng Kết: Áp Dụng Cho NestJS Microservices

| Vấn đề | Instagram Solution | NestJS Implementation |
|---|---|---|
| **Fan-out Feed** | Hybrid push/pull | Bull queue + Redis Sorted Set |
| **Thundering Herd** | Promise caching | `cache-manager` + distributed lock (`redlock`) |
| **Hot Counter** | Sharded counters + Kafka | Redis `INCR` + Kafka consumer batch writes |
| **Connection Pool** | PgBouncer | TypeORM pool config + `pg-pool` |
| **Read/Write Split** | Leader + Replicas | TypeORM `replication` option |
| **Async Pipeline** | Celery + Kafka | `@nestjs/bull` + `@nestjs/microservices` Kafka |
| **Rate Limiting** | Multi-layer token bucket | `@nestjs/throttler` + Redis store |
| **Circuit Breaker** | Custom | `opossum` hoặc `mollitia` |
| **Bulkhead** | Thread pool isolation | Async worker pools + queue partitioning |

> [!TIP]
> **Nguyên tắc #1 của Instagram**: *"No synchronous unbounded operation on the critical path"* — Mọi operation có thể grow theo popularity phải được xử lý async.
