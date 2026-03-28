# URL Shortener - Deployment & Architecture

> Bitly xử lý **28B+ clicks/tháng**, TinyURL phục vụ **1B+ links** — classic system design.

---

## 1. Quy Mô (Bitly Reference)

| Metric | Giá trị |
|---|---|
| Links created | Billions |
| Clicks/month | 28B+ |
| Read:Write ratio | 100:1 (read-heavy) |
| Redirect latency | < 10ms (target) |
| Data per link | ~500 bytes metadata |
| Retention | Permanent (never expire) |

---

## 2. Core Architecture

```mermaid
graph TB
    subgraph "Client"
        WEB9["🌐 Website"]
        API5["📡 API"]
        SDK4["📦 SDKs"]
    end

    subgraph "Edge"
        CDN10["🌍 CDN<br/>(Cache hot redirects)"]
        LB8["⚖️ Load Balancer"]
        RL["🚦 Rate Limiter"]
    end

    subgraph "Application"
        WRITE_SVC["✍️ Write Service<br/>(Create short URL)"]
        READ_SVC["🔀 Redirect Service<br/>(Resolve + redirect)"]
        ANALYTICS_SVC["📊 Analytics Service"]
    end

    subgraph "Data"
        CACHE3["🔴 Redis Cache<br/>(Hot URLs)"]
        DB2["📀 NoSQL DB<br/>(Cassandra/DynamoDB)"]
        ANALYTICS_DB["📊 Analytics Store<br/>(ClickHouse)"]
    end

    subgraph "Async"
        QUEUE5["📨 Kafka<br/>(Click events)"]
    end

    WEB9 & API5 --> CDN10 --> LB8
    LB8 --> RL
    RL --> WRITE_SVC & READ_SVC
    READ_SVC --> CACHE3 --> DB2
    READ_SVC --> QUEUE5 --> ANALYTICS_SVC --> ANALYTICS_DB
```

---

## 3. Data Model

```mermaid
erDiagram
    SHORT_URL {
        string short_code PK "abc123"
        string long_url "https://example.com/very/long/path"
        string user_id FK "Optional: owner"
        timestamp created_at
        timestamp expires_at "Optional"
        string custom_alias "Optional"
    }

    CLICK_EVENT {
        string event_id PK
        string short_code FK
        timestamp clicked_at
        string ip_address
        string user_agent
        string referer
        string country
        string device_type
    }
```

---

## 4. ID Generation Strategies

```mermaid
flowchart TB
    subgraph "Strategy 1: Counter + Base62 (Recommended)"
        COUNTER["Snowflake/Distributed Counter<br/>→ Unique integer"]
        COUNTER --> BASE62["Base62 encode:<br/>1000000 → '4c92'"]
        BASE62 --> SHORT2["✅ short.ly/4c92<br/>(No collision ever!)"]
    end

    subgraph "Strategy 2: Hash (MD5/SHA)"
        LONG["Long URL"] --> HASH4["MD5/SHA256"]
        HASH4 --> TAKE["Take first 7 chars"]
        TAKE --> CHECK8{"Collision?"}
        CHECK8 -->|"No"| STORE3["✅ Store"]
        CHECK8 -->|"Yes"| REHASH["Rehash with salt"]
        REHASH --> TAKE
    end

    subgraph "Strategy 3: Pre-generated Pool"
        POOL2["Pre-generate millions of IDs<br/>Store in Redis"]
        POOL2 --> POP["POP one per request<br/>(O(1), no collision)"]
    end

    style SHORT2 fill:#51cf66,color:#fff
```

### Base62 Capacity

| Length | Possible URLs | Example |
|---|---|---|
| 6 chars | 62⁶ = 56.8 billion | `abc123` |
| 7 chars | 62⁷ = 3.5 trillion | `abc1234` |
| 8 chars | 62⁸ = 218 trillion | `abc12345` |

**7 characters** is the sweet spot — 3.5 trillion unique URLs.

---

## 5. Read Path — Redirect Flow

```mermaid
sequenceDiagram
    actor User as 👤 User
    participant CDN11 as CDN
    participant App as Redirect Service
    participant Cache4 as Redis Cache
    participant DB3 as Database

    User->>CDN11: GET short.ly/abc123

    alt CDN Cache HIT
        CDN11-->>User: 302 → long URL
    else CDN Cache MISS
        CDN11->>App: Forward request

        App->>Cache4: GET abc123
        alt Redis Cache HIT
            Cache4-->>App: long URL
        else Redis Cache MISS
            App->>DB3: SELECT * WHERE short_code = 'abc123'
            DB3-->>App: long URL
            App->>Cache4: SET abc123 → long URL (TTL 24h)
        end

        App-->>User: 302 → long URL
        App->>App: Async: publish click event to Kafka
    end
```

---

## Mapping → NestJS

| Component | NestJS Implementation |
|---|---|
| **Write service** | `POST /api/shorten` controller |
| **Redirect service** | `GET /:code` controller → 302 redirect |
| **ID generation** | Snowflake → `snowflake-id` npm + Base62 |
| **Cache** | `@nestjs/cache-manager` + ioredis |
| **Database** | DynamoDB / PostgreSQL / Cassandra |
| **Rate limiting** | `@nestjs/throttler` |
| **Click events** | `@nestjs/microservices` Kafka producer |
