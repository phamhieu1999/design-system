# Kiến Trúc Hệ Thống Instagram - System Flow

## 1. Tổng Quan Kiến Trúc Toàn Hệ Thống

```mermaid
graph TB
    subgraph "Client Layer"
        iOS["📱 iOS App"]
        Android["📱 Android App"]
        Web["🌐 Web App"]
    end

    subgraph "Edge Layer"
        CDN["🌍 CDN<br/>CloudFlare / Meta CDN<br/>99% media served"]
        LB["⚖️ Load Balancer<br/>L4/L7 + DNS Routing"]
    end

    subgraph "API Gateway"
        GW["🚪 API Gateway<br/>Auth, Rate Limiting,<br/>Request Routing"]
    end

    subgraph "Application Services"
        AUTH["🔐 Auth Service"]
        FEED["📰 Feed Service"]
        MEDIA["📸 Media Service"]
        USER["👤 User Service"]
        STORY["📖 Story Service"]
        REEL["🎬 Reels Service"]
        MSG["💬 DM Service"]
        NOTIF["🔔 Notification Service"]
        SEARCH["🔍 Search Service"]
        RECOM["🤖 Recommendation<br/>ML Service"]
    end

    subgraph "Async Processing"
        KAFKA["📨 Apache Kafka<br/>Event Streaming"]
        CELERY["⚙️ Celery Workers<br/>Background Tasks"]
        RABBIT["🐰 RabbitMQ<br/>Task Queue"]
    end

    subgraph "Data Layer"
        PG["🐘 PostgreSQL<br/>Sharded by user_id<br/>Leader-Follower"]
        CASS["📊 Cassandra<br/>Feeds, Activity Logs"]
        REDIS["🔴 Redis<br/>Cache + Sessions"]
        MEMC["💾 Memcached<br/>Hot Data Cache"]
        ES["🔎 Elasticsearch<br/>Search Index"]
        TAO["🕸️ TAO<br/>Social Graph Store"]
    end

    subgraph "Storage"
        BLOB["📦 Blob Storage<br/>Haystack / Media"]
        ML_STORE["🧠 ML Model Store<br/>Feature Store"]
    end

    iOS & Android & Web --> CDN
    iOS & Android & Web --> LB
    LB --> GW
    GW --> AUTH & FEED & MEDIA & USER & STORY & REEL & MSG & NOTIF & SEARCH & RECOM

    FEED & MEDIA & STORY & REEL --> KAFKA
    KAFKA --> CELERY
    CELERY --> RABBIT
    NOTIF --> KAFKA

    AUTH & USER --> PG
    FEED --> CASS
    MEDIA --> BLOB
    SEARCH --> ES
    USER --> TAO
    RECOM --> ML_STORE

    AUTH & FEED & MEDIA & USER --> REDIS
    AUTH & FEED & MEDIA & USER --> MEMC
```

---

## 2. Luồng Upload Ảnh/Video

```mermaid
sequenceDiagram
    actor User as 📱 User
    participant GW as API Gateway
    participant Media as Media Service
    participant Auth as Auth Service
    participant Blob as Blob Storage
    participant Worker as Celery Worker
    participant Kafka as Kafka
    participant Feed as Feed Service
    participant CDN as CDN
    participant Notif as Notification
    participant Cache as Redis/Memcached

    User->>GW: POST /media/upload (image + metadata)
    GW->>Auth: Verify JWT Token
    Auth-->>GW: ✅ Authenticated

    GW->>Media: Forward upload request
    Media->>Blob: Store original media
    Blob-->>Media: media_id + URL

    Media->>Kafka: Emit EVENT: media.uploaded
    Media-->>User: 202 Accepted (media_id)

    par Background Processing
        Kafka->>Worker: Consume media.uploaded
        Worker->>Worker: Generate thumbnails (150x150, 320x320, 640x640)
        Worker->>Worker: Apply filters / compression
        Worker->>Worker: Extract EXIF, detect content (AI)
        Worker->>Blob: Store processed versions
        Worker->>CDN: Push to CDN edge nodes
    end

    par Feed Distribution
        Kafka->>Feed: Consume media.uploaded
        Feed->>Feed: Fan-out to followers' feeds
        Feed->>Cache: Update feed cache
    end

    par Notifications
        Kafka->>Notif: Consume media.uploaded
        Notif->>Notif: Push notifications to followers
    end
```

---

## 3. Luồng Load Feed (News Feed)

```mermaid
sequenceDiagram
    actor User as 📱 User
    participant GW as API Gateway
    participant Feed as Feed Service
    participant Cache as Redis
    participant Cass as Cassandra
    participant Recom as ML Recommendation
    participant CDN as CDN
    participant PG as PostgreSQL

    User->>GW: GET /feed?cursor=xxx
    GW->>Feed: Get user feed

    Feed->>Cache: Check feed cache
    alt Cache HIT
        Cache-->>Feed: Cached feed data
    else Cache MISS
        Feed->>Cass: Query pre-computed feed
        Cass-->>Feed: Raw feed items
        Feed->>Cache: Store in cache (TTL: 5min)
    end

    Feed->>Recom: Rank & personalize feed
    Recom->>Recom: Two Towers Neural Network
    Recom-->>Feed: Ranked feed items

    Feed->>PG: Fetch user metadata (batch)
    PG-->>Feed: User profiles, like counts

    Feed-->>User: Feed response (JSON)
    
    Note over User,CDN: Media URLs point to CDN
    User->>CDN: Load images/videos from nearest edge
    CDN-->>User: Media content (< 50ms latency)
```

---

## 4. Luồng Deployment (CI/CD)

```mermaid
flowchart TB
    subgraph "Developer"
        A["👨‍💻 Engineer<br/>Push to master"]
    end

    subgraph "CI Pipeline"
        B["🔨 Build System"]
        C["🧪 Automated Tests<br/>Unit + Integration + E2E"]
        D{{"✅ All Tests Pass?"}}
    end

    subgraph "CD Pipeline"
        E["📦 Create Deploy Artifact"]
        F["🐤 Canary Deploy<br/>1-2% servers"]
        G["📊 Sauron Dashboard<br/>Monitor metrics"]
        H{{"📈 Metrics OK?"}}
        I["🔄 Gradual Rollout<br/>10% → 25% → 50%"]
        J{{"📈 Still OK?"}}
        K["🚀 Full Rollout 100%"]
    end

    subgraph "Safety"
        L["⏪ Auto Rollback"]
        M["🚩 Feature Flags<br/>Kill switch"]
    end

    A --> B --> C --> D
    D -->|No| L
    D -->|Yes| E --> F --> G --> H
    H -->|No| L
    H -->|Yes| I --> J
    J -->|No| L
    J -->|Yes| K
    K -.-> M

    style L fill:#ff6b6b,color:#fff
    style K fill:#51cf66,color:#fff
    style F fill:#ffd43b,color:#333
```

---

## 5. Luồng Database Sharding

```mermaid
graph TB
    subgraph "Application"
        APP["Django App"]
        ROUTER["Shard Router<br/>hash(user_id) % N"]
    end

    subgraph "Shard 0 (user_id % 4 = 0)"
        L0["Leader 0 (Write)"]
        R0a["Replica 0a (Read)"]
        R0b["Replica 0b (Read)"]
        L0 --> R0a & R0b
    end

    subgraph "Shard 1 (user_id % 4 = 1)"
        L1["Leader 1 (Write)"]
        R1a["Replica 1a (Read)"]
        R1b["Replica 1b (Read)"]
        L1 --> R1a & R1b
    end

    subgraph "Shard 2 (user_id % 4 = 2)"
        L2["Leader 2 (Write)"]
        R2a["Replica 2a (Read)"]
        R2b["Replica 2b (Read)"]
        L2 --> R2a & R2b
    end

    subgraph "Shard 3 (user_id % 4 = 3)"
        L3["Leader 3 (Write)"]
        R3a["Replica 3a (Read)"]
        R3b["Replica 3b (Read)"]
        L3 --> R3a & R3b
    end

    APP --> ROUTER
    ROUTER --> L0 & L1 & L2 & L3
    ROUTER -.->|Reads| R0a & R1a & R2a & R3a

    style L0 fill:#ff922b,color:#fff
    style L1 fill:#ff922b,color:#fff
    style L2 fill:#ff922b,color:#fff
    style L3 fill:#ff922b,color:#fff
```

---

## 6. Luồng Caching Multi-Layer

```mermaid
flowchart LR
    A["📱 Client Request"] --> B{"🌍 CDN Cache"}
    B -->|HIT| C["⚡ Return (< 10ms)"]
    B -->|MISS| D{"💾 Memcached"}
    D -->|HIT| E["⚡ Return (< 1ms)"]
    D -->|MISS| F{"🔴 Redis"}
    F -->|HIT| G["⚡ Return (< 2ms)"]
    F -->|MISS| H["🐘 PostgreSQL / Cassandra"]
    H --> I["Return (10-50ms)"]
    I --> J["Write back to Redis"]
    J --> K["Write back to Memcached"]

    style C fill:#51cf66,color:#fff
    style E fill:#51cf66,color:#fff
    style G fill:#51cf66,color:#fff
    style I fill:#ffd43b,color:#333
```

---

## 7. Luồng Real-time (DM & Notifications)

```mermaid
sequenceDiagram
    actor UserA as 👤 User A
    participant GW as API Gateway
    participant DM as DM Service
    participant Kafka as Kafka
    participant WS as WebSocket Server
    participant Notif as Notification Service
    participant Push as Push Gateway<br/>(APNs / FCM)
    actor UserB as 👤 User B

    UserA->>GW: POST /dm/send (to: UserB, text: "Hi!")
    GW->>DM: Forward message
    DM->>DM: Encrypt & store message
    DM->>Kafka: Emit EVENT: dm.sent

    par Real-time delivery
        Kafka->>WS: Consume dm.sent
        WS->>UserB: WebSocket push (if online)
    end

    par Push notification
        Kafka->>Notif: Consume dm.sent
        alt User B is offline
            Notif->>Push: Send push notification
            Push->>UserB: 🔔 "User A sent you a message"
        end
    end

    DM-->>UserA: ✅ Message sent
```

---

## 8. Geo-Distribution & Data Center

```mermaid
graph TB
    subgraph "US West"
        DC1["🏢 Data Center 1<br/>Primary"]
        DC1_PG["PostgreSQL Leader"]
        DC1_CASS["Cassandra Node"]
        DC1 --- DC1_PG & DC1_CASS
    end

    subgraph "US East"
        DC2["🏢 Data Center 2"]
        DC2_PG["PostgreSQL Replica"]
        DC2_CASS["Cassandra Node"]
        DC2 --- DC2_PG & DC2_CASS
    end

    subgraph "Europe"
        DC3["🏢 Data Center 3"]
        DC3_PG["PostgreSQL Replica"]
        DC3_CASS["Cassandra Node"]
        DC3 --- DC3_PG & DC3_CASS
    end

    subgraph "Asia"
        DC4["🏢 Data Center 4"]
        DC4_PG["PostgreSQL Replica"]
        DC4_CASS["Cassandra Node"]
        DC4 --- DC4_PG & DC4_CASS
    end

    DC1_PG -->|Async Replication| DC2_PG & DC3_PG & DC4_PG
    DC1_CASS <-->|Peer-to-Peer| DC2_CASS & DC3_CASS & DC4_CASS

    DNS["🌐 DNS<br/>GeoDNS Routing"] -->|Nearest DC| DC1 & DC2 & DC3 & DC4

    style DC1 fill:#4c6ef5,color:#fff
    style DC2 fill:#7950f2,color:#fff
    style DC3 fill:#ae3ec9,color:#fff
    style DC4 fill:#e64980,color:#fff
```
