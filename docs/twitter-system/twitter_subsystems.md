# Twitter/X - Subsystems Analysis

> Search, Storage, Real-time, Data Pipeline, Reliability.

---

## 1. Search — Earlybird + Elasticsearch

### 1.1 Real-time Tweet Search (Earlybird)

```mermaid
flowchart TB
    TWEET2["📝 New Tweet"] --> KAFKA10["Kafka"]
    KAFKA10 --> INDEX["Earlybird Indexer<br/>(Custom Lucene)"]

    subgraph "Earlybird Cluster"
        INDEX --> SHARD_A["Shard A<br/>(tweet_id hash)"]
        INDEX --> SHARD_B["Shard B"]
        INDEX --> SHARD_C["Shard C"]
    end

    QUERY["🔍 Search Query"] --> SCATTER["Scatter request<br/>to all shards"]
    SCATTER --> SHARD_A & SHARD_B & SHARD_C
    SHARD_A & SHARD_B & SHARD_C --> GATHER["Gather & merge results"]
    GATHER --> RANK3["ML Rank<br/>(relevance + recency)"]
    RANK3 --> RESULTS2["📋 Results"]
```

| Feature | Earlybird | Elasticsearch |
|---|---|---|
| **Use** | Real-time tweet search | Users, DMs, secondary |
| **Indexing** | Lock-free, single-writer | Standard Lucene |
| **Latency** | < 50ms (in-memory) | < 100ms |
| **Tweets** | Index trong vài giây | Batch/near-real-time |

### 1.2 Trending Topics Detection

```mermaid
flowchart LR
    STREAM["Tweet Stream<br/>500K/min"] --> EXTRACT2["Extract entities<br/>(hashtags, keywords, ngrams)"]
    EXTRACT2 --> WINDOW2["Sliding Windows<br/>5m | 15m | 1h"]
    WINDOW2 --> COMPARE["Compare vs baseline<br/>(same time yesterday/last week)"]
    COMPARE --> SCORE["Score = velocity × volume<br/>normalized by geo/language"]
    SCORE --> FILTER2["Filter spam/bots"]
    FILTER2 --> TOP["🏆 Top 10 Trending"]
```

**Key:** Trending = **tốc độ tăng bất thường**, không phải tổng count cao nhất.

---

## 2. Storage — Manhattan

```mermaid
graph TB
    subgraph "Manhattan Architecture"
        CLIENT["Service Client"] --> COORDINATOR["Coordinator<br/>(Routing + Replication)"]

        COORDINATOR --> NODE1["Storage Node 1"]
        COORDINATOR --> NODE2["Storage Node 2"]
        COORDINATOR --> NODE3["Storage Node 3"]

        NODE1 --- ENGINE1["RocksDB<br/>(Write-heavy)"]
        NODE2 --- ENGINE2["B-Tree<br/>(Read-heavy)"]
        NODE3 --- ENGINE3["SeaDB<br/>(Read-only, batch)"]
    end

    subgraph "Data Model"
        KEY["Key = P-Key + L-Key"]
        KEY --- EX1["P-Key: user_id<br/>L-Key: tweet_id"]
        KEY --- EX2["P-Key: tweet_id<br/>L-Key: 'metadata'"]
    end

    subgraph "Consistency Modes"
        EC["Eventually Consistent<br/>(Default, high availability)"]
        SC["Strongly Consistent<br/>(Opt-in, mastership)"]
        RO["Read-only<br/>(Batch data from HDFS)"]
    end
```

### Manhattan vs Instagram PostgreSQL

| Aspect | Manhattan (Twitter) | PostgreSQL (Instagram) |
|---|---|---|
| **Type** | Distributed KV store | Relational (sharded) |
| **Query** | Key-based lookups | SQL (joins, aggregates) |
| **Consistency** | Eventually consistent (default) | Strong (ACID) |
| **Sharding** | Built-in consistent hashing | Application-level by user_id |
| **Use case** | Tweets, DMs, user data | Profiles, relationships, metadata |

---

## 3. Real-time — Streaming & Notifications

### 3.1 Notification Priority System

```mermaid
flowchart TB
    EVENT["🎯 Event Occurs<br/>(mention, like, retweet, follow)"] --> KAFKA11["Kafka"]

    KAFKA11 --> PRIORITY{"Priority?"}

    PRIORITY -->|"P0: DM, Security"| INSTANT["⚡ Instant Delivery<br/>(< 100ms)"]
    PRIORITY -->|"P1: Mention, Reply"| FAST["🏃 Fast Delivery<br/>(< 5s)"]
    PRIORITY -->|"P2: Like, Retweet"| BATCH["📦 Batched<br/>(30s window)"]
    PRIORITY -->|"P3: Suggestion"| DEFER["⏰ Deferred<br/>(1h-24h)"]

    INSTANT --> APNS_FCM["Push Gateway<br/>(APNs/FCM)"]
    FAST --> APNS_FCM
    BATCH --> AGGREGATE2["Aggregate:<br/>'X and 42 others liked'"]
    AGGREGATE2 --> APNS_FCM

    style INSTANT fill:#ff6b6b,color:#fff
    style FAST fill:#ffd43b,color:#333
    style BATCH fill:#4c6ef5,color:#fff
```

### 3.2 Streaming API (Firehose)

```mermaid
sequenceDiagram
    participant Client as API Consumer
    participant LB as Load Balancer
    participant Stream as Streaming Server
    participant Kafka as Kafka (All tweets)

    Client->>LB: POST /stream?track=bitcoin,crypto
    LB->>Stream: Assign to server
    Stream->>Kafka: Subscribe to tweet stream

    loop Persistent Connection
        Kafka->>Stream: New tweet published
        Stream->>Stream: Match against filters
        alt Matches filter
            Stream->>Client: Push tweet JSON (chunked HTTP)
        end
    end

    Note over Client,Kafka: Connection stays open indefinitely
```

| API Tier | Data | Use Case |
|---|---|---|
| **Firehose** | 100% public tweets | Data partners, research |
| **Filter stream** | Filtered by keyword/user/geo | App developers |
| **Sample stream** | ~1% random sample | Analytics, trend detection |

---

## 4. Data Pipeline

```mermaid
flowchart LR
    subgraph "Sources"
        S1["Tweets"] & S2["Engagements"] & S3["Ads clicks"] & S4["Server logs"]
    end

    subgraph "Ingestion"
        S1 & S2 & S3 & S4 --> KAFKA12["Kafka"]
    end

    subgraph "Processing"
        KAFKA12 --> STORM["Apache Storm / Heron<br/>(Real-time)"]
        KAFKA12 --> SPARK2["Apache Spark<br/>(Batch)"]
    end

    subgraph "Storage"
        STORM --> HDFS3["HDFS / Google Cloud Storage"]
        SPARK2 --> HDFS3
    end

    subgraph "Query"
        HDFS3 --> PRESTO2["Presto<br/>(Interactive)"]
        HDFS3 --> HIVE2["Hive<br/>(Batch reports)"]
    end
```

**Twitter → Heron:** Twitter từng dùng Apache Storm nhưng sau đó build **Heron** (open-source) để thay thế vì performance và debugging tốt hơn.

---

## 5. Reliability & SRE

```mermaid
flowchart TB
    subgraph "Resiliency Patterns"
        CB2["🔌 Circuit Breaker<br/>(Finagle built-in)"]
        RETRY["🔄 Retry + Backoff<br/>(Finagle built-in)"]
        TIMEOUT["⏱️ Timeouts<br/>(Strict per-service)"]
        LB3["⚖️ Load Balancing<br/>(P2C - Power of 2 Choices)"]
        SHED["🚫 Load Shedding<br/>(Drop low-priority under load)"]
    end

    subgraph "Monitoring"
        METRICS["📊 Metrics<br/>(Observability platform)"]
        TRACE["🔍 Distributed Tracing<br/>(Zipkin - Twitter created)"]
        LOG["📋 Structured Logging"]
    end

    subgraph "Failure Handling"
        DEGRADE["📉 Graceful Degradation:<br/>1. Disable trending<br/>2. Simplified timeline<br/>3. Cached search results"]
    end
```

**Fun fact:** Twitter created **Zipkin** — hệ thống distributed tracing open-source phổ biến nhất, dựa trên Google Dapper paper.

### Twitter/X Unique Innovations

| Innovation | Mô tả | Impact |
|---|---|---|
| **Snowflake** | Distributed ID generator | Industry standard (Discord, Instagram dùng variant) |
| **Finagle** | Async RPC framework (Scala) | Foundation cho microservices |
| **Zipkin** | Distributed tracing | De-facto standard, integrated vào Spring |
| **Heron** | Stream processing (Storm replacement) | 10x throughput vs Storm |
| **Manhattan** | Multi-tenant KV store | Replace Cassandra, MySQL cho many use cases |
| **Earlybird** | Real-time search engine | Sub-second tweet indexing |

---

## So Sánh Tổng Hợp: Twitter vs Instagram

| Dimension | Twitter/X | Instagram |
|---|---|---|
| **Language** | Scala/Java (JVM) | Python (Django) |
| **RPC** | Finagle (custom) | Thrift / internal |
| **Primary DB** | Manhattan (KV) | PostgreSQL (relational) |
| **Timeline cache** | Redis Sorted Sets | Redis + Cassandra |
| **Search** | Earlybird (custom Lucene) | Unicorn → Vector search |
| **ID system** | Snowflake (custom 64-bit) | PostgreSQL sequences |
| **Orchestration** | Kubernetes (GKE) | Tupperware (Meta internal) |
| **Cloud** | Google Cloud | Meta own DCs |
| **Tracing** | Zipkin (created by Twitter) | Internal tools |
| **Stream processing** | Heron / Storm | Flink |
