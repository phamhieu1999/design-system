# Twitter/X - Xử Lý Đồng Thời Cao & Timeline Fan-out

> Phân tích cách Twitter xử lý 500K+ tweets/phút và deliver tới 500M+ users.

---

## 1. Timeline Fan-out — Hybrid Push/Pull

Đây là **core engineering challenge** lớn nhất của Twitter.

```mermaid
flowchart TB
    TWEET["📝 User posts tweet"] --> CHECK2{"👤 Follower count?"}

    CHECK2 -->|"< 100K followers<br/>(99.9% users)"| PUSH2["📤 Fan-out on WRITE"]
    CHECK2 -->|"> 100K followers<br/>(Celebrities)"| PULL2["📥 Fan-out on READ"]

    subgraph "Push Path (Write time)"
        PUSH2 --> KAFKA7["Kafka: tweet.created"]
        KAFKA7 --> WORKERS2["Fan-out Workers<br/>(parallel, batched)"]
        WORKERS2 --> REDIS3["Redis: timeline:{follower_id}<br/>LPUSH tweet_id (sorted set)"]
    end

    subgraph "Pull Path (Read time)"
        PULL2 --> STORE2["Manhattan: Store tweet once"]
    end

    subgraph "User Opens Timeline"
        OPEN2["📱 User opens app"]
        OPEN2 --> MERGE3["Get cached timeline (Redis)"]
        OPEN2 --> FETCH["Fetch celebrity tweets on-the-fly"]
        MERGE3 --> MERGE4["🔀 Merge + ML Rank"]
        FETCH --> MERGE4
        MERGE4 --> DISPLAY["📰 Home Timeline"]
    end

    style PUSH2 fill:#51cf66,color:#fff
    style PULL2 fill:#ffd43b,color:#333
    style MERGE4 fill:#4c6ef5,color:#fff
```

### Fan-out Numbers

| Scenario | Writes triggered | Time to deliver |
|---|---|---|
| User with 100 followers | 100 Redis LPUSH | < 5 seconds |
| User with 10K followers | 10K Redis LPUSH | < 10 seconds |
| Celebrity with 50M followers | **0** writes (pull model) | On-demand merge |

---

## 2. Tweet Write Path

```mermaid
sequenceDiagram
    actor User as 👤 User
    participant API as API Gateway
    participant Tweet as Tweet Service
    participant Snow as Snowflake
    participant MH as Manhattan
    participant Kafka as Kafka
    participant Fanout as Fan-out Service
    participant Redis as Redis Cache
    participant Search as Earlybird

    User->>API: POST /tweet {"text": "Hello!"}
    API->>Tweet: Create tweet
    Tweet->>Snow: Generate Snowflake ID
    Snow-->>Tweet: id: 1912345678901234

    Tweet->>MH: Store tweet (K: tweet_id, V: tweet_data)
    Tweet->>Kafka: Emit tweet.created event
    Tweet-->>User: 200 OK {id, text, timestamp}

    par Fan-out (async)
        Kafka->>Fanout: Consume tweet.created
        Fanout->>Fanout: Lookup followers (User Graph)
        Fanout->>Redis: LPUSH timeline:{follower_id} tweet_id
        Note over Fanout,Redis: Batched — 4K followers/batch
    end

    par Search indexing (async)
        Kafka->>Search: Index tweet in Earlybird
        Note over Search: Searchable within seconds
    end
```

---

## 3. Tweet Read Path (Home Timeline)

```mermaid
sequenceDiagram
    actor User as 👤 User
    participant API as API Gateway
    participant TL as Timeline Service
    participant Redis as Redis Timeline Cache
    participant MH as Manhattan
    participant Rank as ML Ranker

    User->>API: GET /home_timeline?cursor=xxx
    API->>TL: Fetch timeline

    TL->>Redis: GET timeline:{user_id} (top 800 tweet_ids)
    Redis-->>TL: [id1, id2, id3, ..., id800]

    TL->>MH: Multi-GET tweet details (batch)
    MH-->>TL: [{id1, text, author...}, ...]

    par Celebrity tweets (pull)
        TL->>MH: Fetch recent tweets from celebrity follows
        MH-->>TL: Celebrity tweets
    end

    TL->>Rank: Rank all candidates
    Rank->>Rank: ML model (engagement prediction)
    Rank-->>TL: Ranked timeline

    TL-->>User: Timeline JSON (top 20-50 tweets)
```

---

## 4. Thundering Herd — Celebrity Tweet Spike

Khi Elon tweet → hàng triệu users mở app cùng lúc.

```mermaid
flowchart TB
    SPIKE["📈 Celebrity tweets<br/>→ Millions open app"] --> CACHE_HOT["🔥 Hot key problem<br/>timeline:{celebrity} gets millions reads"]

    subgraph "Solutions"
        S1["1️⃣ Redis Cluster<br/>Shard across 100+ nodes"]
        S2["2️⃣ Local Cache (L1)<br/>Each app server caches 1s"]
        S3["3️⃣ Request Coalescing<br/>Deduplicate identical reqs"]
        S4["4️⃣ Stale-while-revalidate<br/>Serve stale, refresh async"]
    end

    CACHE_HOT --> S1 & S2 & S3 & S4

    style SPIKE fill:#ff6b6b,color:#fff
```

---

## 5. Trending Topics — Real-time Detection

```mermaid
flowchart TB
    TWEETS["📝 500K tweets/min"] --> KAFKA8["Kafka Stream"]

    KAFKA8 --> EXTRACT["Extract:<br/>• Hashtags<br/>• Keywords<br/>• Entities"]
    EXTRACT --> WINDOW["⏰ Sliding Window<br/>(5 min, 1 hour)"]
    WINDOW --> COUNT["📊 Count frequencies"]
    COUNT --> DETECT["🤖 Anomaly Detection<br/>current_count vs baseline"]

    DETECT --> TRENDING{"Spike detected?"}
    TRENDING -->|Yes| RANK2["🏆 Rank by:<br/>• Velocity (tốc độ tăng)<br/>• Volume (tổng count)<br/>• Geography"]
    TRENDING -->|No| IGNORE["Skip"]
    RANK2 --> DISPLAY2["📱 Trending Topics"]

    style DETECT fill:#4c6ef5,color:#fff
```

**Key insight:** Trending KHÔNG phải top hashtags theo tổng count. Mà là hashtags có **tốc độ tăng bất thường** — velocity > baseline.

---

## 6. Engagement Counters — Like/Retweet

Giống Instagram celebrity like problem, Twitter dùng sharded counters.

```mermaid
flowchart LR
    LIKE["❤️ Like tweet"] --> KAFKA9["Kafka: engagement.like"]
    KAFKA9 --> REDIS_INC["Redis INCR<br/>tweet:{id}:likes<br/>(instant UI)"]
    KAFKA9 --> WORKER3["Workers"]
    WORKER3 --> SHARD1["Shard 0: +N"]
    WORKER3 --> SHARD2["Shard 1: +N"]
    WORKER3 --> SHARD3["Shard 2: +N"]
    SHARD1 & SHARD2 & SHARD3 --> AGG2["⏰ Aggregate → Manhattan"]

    style REDIS_INC fill:#51cf66,color:#fff
```

---

## So Sánh: Twitter vs Instagram Fan-out

| Aspect | Twitter | Instagram |
|---|---|---|
| **Content type** | Text-first (280 chars) | Media-first (photos/videos) |
| **Fan-out threshold** | ~100K followers | ~500K followers |
| **Timeline cache** | Redis (Sorted Sets) | Redis + Cassandra |
| **Storage** | Manhattan (custom KV) | PostgreSQL (sharded) |
| **Search** | Earlybird (custom Lucene) | Unicorn (graph-aware) |
| **ID generation** | Snowflake (custom) | PostgreSQL sequences (sharded) |
| **Celebrity merge** | On-read, merge 2 sources | On-read, merge 2 sources |

---

## Mapping → NestJS

| Pattern | Twitter/X | NestJS Implementation |
|---|---|---|
| **Fan-out** | Kafka → Redis LPUSH (batched) | `@nestjs/microservices` Kafka → Bull job → Redis |
| **Timeline cache** | Redis Sorted Set | `ioredis` ZADD/ZRANGE |
| **Trending** | Sliding window + anomaly detect | Redis Sorted Set + cron job frequency analysis |
| **Sharded counter** | Kafka → multi-shard workers | BullMQ + Redis INCR + periodic DB flush |
| **Snowflake ID** | Custom 64-bit generator | `snowflake-id` npm package / custom BigInt |
| **Request coalescing** | Promise-based dedup | `dataloader` pattern (batch + cache) |
