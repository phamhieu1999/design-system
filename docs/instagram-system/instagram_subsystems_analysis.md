# Phân Tích Các Subsystem Còn Lại - Instagram

> 7 khía cạnh: Media Processing, Social Graph, Data Pipeline, ML/AI, Search, Real-time, Reliability.

---

## 1. Media Processing Pipeline

Instagram xử lý **500+ triệu uploads/ngày** qua pipeline DAG (Directed Acyclic Graph) async.

```mermaid
flowchart TB
    subgraph "Upload Path (< 200ms response)"
        UPLOAD["📱 User Upload"] --> PRESIGN["Pre-signed URL<br/>Upload trực tiếp → Object Storage"]
        PRESIGN --> BLOB["📦 Raw Blob Storage"]
        BLOB --> KAFKA["📨 Kafka: media.uploaded"]
        KAFKA --> ACK["✅ 202 Accepted → User"]
    end

    subgraph "DAG Processing Pipeline (Async)"
        KAFKA --> DAG["🔀 DAG Scheduler"]
        
        DAG --> V1["🎬 Video Transcoding<br/>240p → 480p → 720p → 1080p"]
        DAG --> V2["🖼️ Thumbnail Generation<br/>150x150, 320x320, 640x640"]
        DAG --> V3["🤖 AI Content Analysis<br/>NSFW, CSAM, Objects"]
        DAG --> V4["📝 EXIF Extraction<br/>Metadata, GPS, Camera"]
        DAG --> V5["🏷️ Auto-tagging<br/>Scene, Objects, Text"]
        
        V1 --> STORE["📦 Processed Storage"]
        V2 --> STORE
        V3 --> POLICY{"Policy OK?"}
        POLICY -->|No| BLOCK["🚫 Block"]
        POLICY -->|Yes| STORE
    end

    subgraph "Distribution"
        STORE --> CDN["🌍 CDN Push<br/>Edge nodes worldwide"]
    end

    style ACK fill:#51cf66,color:#fff
    style BLOCK fill:#ff6b6b,color:#fff
```

### Adaptive Bitrate Streaming (Reels)

| Resolution | Bitrate | Use Case |
|---|---|---|
| 240p | 300 kbps | 2G/3G, low-end devices |
| 480p | 800 kbps | 4G, mid-range |
| 720p | 2 Mbps | WiFi, standard |
| 1080p | 5 Mbps | WiFi, flagship devices |

**Tối ưu đặc biệt:** Meta giảm **94% compute** bằng cách repackage frames từ encoding sẵn có thay vì transcode lại từ source → tiết kiệm hàng triệu USD/năm.

**Prefetching:** App prefetch 2-3 giây đầu của video tiếp theo → user swipe thấy video play "instant".

---

## 2. Social Graph — TAO

TAO (The Associations and Objects) là distributed graph store phục vụ social graph cho Instagram.

```mermaid
graph TB
    subgraph "Data Model"
        U1["👤 User A<br/>(Object, id=123)"]
        U2["👤 User B<br/>(Object, id=456)"]
        P1["📸 Post<br/>(Object, id=789)"]
        
        U1 -->|"FOLLOWS<br/>(Association)"| U2
        U1 -->|"AUTHORED<br/>(Association)"| P1
        U2 -->|"LIKED<br/>(Association)"| P1
    end

    subgraph "Architecture"
        APP2["Application"] --> CACHE2["Graph-aware Cache<br/>(in-memory)"]
        CACHE2 -->|"Cache Miss"| MYSQL["MySQL<br/>(Persistent, Sharded)"]
        CACHE2 -.->|"Write-through"| MYSQL
    end

    subgraph "Geo-Distribution"
        PRIMARY["🟠 Primary Region<br/>(All Writes)"]
        FOLLOWER1["🔵 Follower Region 1<br/>(Local Reads)"]
        FOLLOWER2["🔵 Follower Region 2<br/>(Local Reads)"]
        
        PRIMARY -->|"Async Replication"| FOLLOWER1 & FOLLOWER2
        FOLLOWER1 -->|"Forward Writes"| PRIMARY
    end
```

| Feature | Mô tả |
|---|---|
| **Objects** | Typed nodes: User, Post, Comment, Story (64-bit ID) |
| **Associations** | Directed edges: FOLLOWS, LIKES, AUTHORED (có timestamp) |
| **Write-through cache** | Update cache ngay khi write → keep cache warm |
| **Sharding** | Hàng trăm nghìn shards trên MySQL |

---

## 3. Data Pipeline & Analytics

```mermaid
flowchart LR
    subgraph "Data Sources"
        S1["📱 App Events"]
        S2["🖥️ Server Logs"]
        S3["🗄️ DB Changes"]
    end

    subgraph "Ingestion (Streaming)"
        S1 & S2 & S3 --> KAFKA4["📨 Kafka / Scribe"]
    end

    subgraph "Processing"
        KAFKA4 --> FLINK["⚡ Flink<br/>(Real-time)"]
        KAFKA4 --> SPARK["🔥 Spark<br/>(Batch)"]
    end

    subgraph "Storage"
        FLINK --> LAKE["🏞️ Data Lake<br/>(HDFS + Iceberg)"]
        SPARK --> LAKE
    end

    subgraph "Query & Analytics"
        LAKE --> HIVE["🐝 Hive<br/>(Batch SQL)"]
        LAKE --> PRESTO["⚡ Presto<br/>(Interactive SQL)"]
    end

    subgraph "Orchestration"
        AIRFLOW["🌬️ Airflow<br/>ETL Scheduling"]
        AIRFLOW --> SPARK
        AIRFLOW --> HIVE
    end

    style FLINK fill:#51cf66,color:#fff
    style PRESTO fill:#4c6ef5,color:#fff
```

| Layer | Technology | Throughput |
|---|---|---|
| **Ingestion** | Kafka + Scribe | Hàng nghìn tỷ events/ngày |
| **Stream Processing** | Apache Flink | Real-time (< 1s latency) |
| **Batch Processing** | Apache Spark | Petabytes/ngày |
| **Data Lake** | HDFS + Apache Iceberg | ACID, time-travel queries |
| **Query** | Presto (interactive), Hive (batch) | Ad-hoc + scheduled |
| **Orchestration** | Airflow / MetaFlow | DAG scheduling, retries |
| **Schema** | Avro / Protobuf | Schema enforcement |

**Deterministic Sampling:** Chỉ xử lý 1% data cho dev/testing → iterate nhanh, giảm chi phí infrastructure.

---

## 4. ML/AI Recommendation

Instagram dùng **multi-stage ranking funnel** để chọn content cho Feed, Explore, Reels.

```mermaid
flowchart TB
    POOL["🌊 Content Pool<br/>Hàng tỷ posts"] --> STAGE1

    subgraph "Stage 1: Retrieval (~1500 candidates)"
        STAGE1["🏗️ Two Towers Model<br/>User Tower ↔ Item Tower"]
        STAGE1A["ANN Search (FAISS)<br/>Nearest neighbor trong vector space"]
        STAGE1 --> STAGE1A
    end

    STAGE1A --> STAGE2

    subgraph "Stage 2: First Ranking (~500)"
        STAGE2["🎯 Lightweight Neural Net<br/>Distilled from Stage 3"]
    end

    STAGE2 --> STAGE3

    subgraph "Stage 3: Second Ranking (~150)"
        STAGE3["🧮 GBDT / Neural Net<br/>Dense + sparse features"]
    end

    STAGE3 --> STAGE4

    subgraph "Stage 4: Final Reranking (~50)"
        STAGE4["🤖 Deep Neural Network<br/>Full feature set"]
        STAGE4A["📏 Business Rules:<br/>• Diversity (không trùng author)<br/>• Policy filtering<br/>• Ads insertion"]
        STAGE4 --> STAGE4A
    end

    STAGE4A --> FEED["📱 User Feed / Explore"]

    style POOL fill:#868e96,color:#fff
    style FEED fill:#51cf66,color:#fff
```

### Two Towers Model Detail

```mermaid
graph LR
    subgraph "👤 User Tower"
        UF["User Features:<br/>• Demographics<br/>• Interest history<br/>• Recent interactions"]
        UF --> UNN["Neural Network"]
        UNN --> UE["User Embedding<br/>(128-dim vector)"]
    end

    subgraph "📸 Item Tower"
        IF["Item Features:<br/>• Content type<br/>• Author info<br/>• Engagement stats"]
        IF --> INN["Neural Network"]
        INN --> IE["Item Embedding<br/>(128-dim vector)"]
    end

    UE --> DOT["Dot Product<br/>Similarity Score"]
    IE --> DOT

    IE -.->|"Pre-computed<br/>& cached offline"| CACHE3["FAISS Index"]
```

| Concept | Mô tả |
|---|---|
| **Teacher Model** | Model lớn, train offline với full features |
| **Student Model** | Model nhẹ, distill từ teacher → serve online |
| **Engagement Signals** | Likes, saves, shares, watch time, comments |
| **Diversity Rules** | Không quá 2 posts liên tiếp từ 1 author |

---

## 5. Search & Discovery

```mermaid
flowchart TB
    subgraph "Evolution"
        ES["2012-2015<br/>Elasticsearch"] --> UNI["2015-nay<br/>Unicorn<br/>(Social-graph-aware)"]
        UNI --> VEC["2020+<br/>Vector Search<br/>(ANN/FAISS)"]
    end

    subgraph "Modern Search Flow"
        QUERY["🔍 User Query"] --> TOKENIZE["Tokenizer<br/>+ Intent Detection"]
        
        TOKENIZE --> KW["📝 Keyword Match<br/>(Users, Hashtags, Places)"]
        TOKENIZE --> EMBED["🧠 Embedding Search<br/>(Semantic similarity)"]
        TOKENIZE --> GRAPH["🕸️ Graph Search<br/>(2nd-degree connections via TAO)"]
        
        KW & EMBED & GRAPH --> MERGE2["🔀 Merge & Rank"]
        MERGE2 --> ML_RANK["🤖 ML Ranking<br/>(Personalized)"]
        ML_RANK --> RESULTS["📋 Search Results"]
    end

    style ES fill:#868e96,color:#fff
    style VEC fill:#51cf66,color:#fff
```

### Search Signals

| Signal | Weight | Ví dụ |
|---|---|---|
| **Text Match** | High | Username, hashtag, caption keywords |
| **Social Graph** | High | Accounts bạn follow, mutual connections |
| **Engagement** | Medium | Popularity, trending score |
| **Recency** | Medium | Nội dung mới hơn được ưu tiên |
| **Embeddings** | Medium | Semantic similarity (visual + text) |
| **User Interest** | Medium | Lịch sử tương tác cá nhân |

---

## 6. Real-time Systems

### 6.1 Messaging & Notifications

```mermaid
sequenceDiagram
    actor A as 👤 User A
    participant GW as API Gateway
    participant MSG as Message Service
    participant REG as Connection Registry
    participant MQTT as MQTT Broker
    participant KAFKA5 as Kafka
    participant NOTIF as Notification Service
    participant PUSH as APNs / FCM
    actor B as 👤 User B

    A->>GW: Send message "Hello!"
    GW->>MSG: Store & process
    MSG->>MSG: Encrypt & persist (Cassandra)
    MSG->>REG: Where is User B?

    alt User B ONLINE
        REG-->>MSG: Server #42, connection active
        MSG->>MQTT: Publish to User B's topic
        MQTT->>B: Real-time delivery (< 100ms)
    else User B OFFLINE
        REG-->>MSG: No active connection
        MSG->>KAFKA5: Emit dm.sent event
        KAFKA5->>NOTIF: Consume event
        NOTIF->>NOTIF: Check preferences & dedup
        NOTIF->>PUSH: Send push notification
        PUSH->>B: 🔔 "User A: Hello!"
    end
```

### 6.2 Live Streaming

```mermaid
flowchart TB
    STREAMER["🎥 Streamer"] --> INGEST["Ingest Server<br/>(RTMP)"]
    INGEST --> TRANSCODE2["Real-time Transcode<br/>(Multiple ABR levels)"]
    TRANSCODE2 --> SEGMENT["Segment (2-3s chunks)<br/>HLS manifest (.m3u8)"]
    SEGMENT --> CDN2["🌍 CDN Edge Nodes"]

    STREAMER --> NOTIFY_LIVE["📨 Kafka: live.started"]
    NOTIFY_LIVE --> FANOUT["Fan-out Service<br/>(Prioritized followers)"]
    FANOUT --> PUSH2["🔔 Push: 'X is live!'"]

    CDN2 --> VIEWER["👁️ Viewers"]
    VIEWER --> INTERACT["💬 Comments, ❤️ Reactions"]
    INTERACT --> WS["WebSocket<br/>(Real-time overlay)"]

    style STREAMER fill:#ff6b6b,color:#fff
    style CDN2 fill:#4c6ef5,color:#fff
```

### Notification Priority & Deduplication

| Priority | Type | Delivery |
|---|---|---|
| **P0 Critical** | DM, security alert | Instant push |
| **P1 High** | Live started, mention | Within 5s |
| **P2 Medium** | Like, follow, comment | Batched (30s window) |
| **P3 Low** | Suggestions, weekly digest | Delayed / email |

**Dedup rules:** Không gửi "X liked your post" 100 lần → aggregate thành "X and 99 others liked your post".

---

## 7. Reliability & SRE

### 7.1 Self-healing Infrastructure

```mermaid
flowchart TB
    FAILURE["⚠️ Failure Detected<br/>(disk, process, network)"] --> FBAR["🤖 FBAR<br/>(Auto-Remediation)"]
    
    FBAR --> DIAG["Diagnose:<br/>• Hardware fault?<br/>• Software crash?<br/>• Network partition?"]
    
    DIAG --> FIX{"Auto-fix possible?"}
    FIX -->|Yes| AUTO["⚡ Auto Remediate:<br/>• Restart process<br/>• Replace disk<br/>• Drain traffic"]
    FIX -->|No| ESCALATE["📟 Page On-call<br/>with full diagnosis"]
    
    AUTO --> VERIFY2["✅ Verify health"]
    VERIFY2 -->|OK| DONE["Resolved<br/>(no human needed)"]
    VERIFY2 -->|Still broken| ESCALATE

    style FBAR fill:#4c6ef5,color:#fff
    style DONE fill:#51cf66,color:#fff
```

### 7.2 Disaster Recovery — DR Storms

```mermaid
flowchart LR
    subgraph "Normal Operation"
        DC_A["🏢 DC-A (33%)"]
        DC_B["🏢 DC-B (33%)"]
        DC_C["🏢 DC-C (33%)"]
    end

    subgraph "DR Storm Test"
        DC_A2["🏢 DC-A (50%)"]
        DC_B2["❌ DC-B (OFFLINE)"]
        DC_C2["🏢 DC-C (50%)"]
    end

    DC_A --> DC_A2
    DC_B --> DC_B2
    DC_C --> DC_C2

    DC_B2 -->|"Traffic rerouted"| DC_A2 & DC_C2

    subgraph "Graceful Degradation"
        DEG["During overload:<br/>1. Disable Explore recommendations<br/>2. Reduce feed refresh rate<br/>3. Queue non-critical notifications<br/>4. Serve cached content"]
    end

    style DC_B2 fill:#ff6b6b,color:#fff
```

### 7.3 Observability Stack

| Layer | Tool | Mục đích |
|---|---|---|
| **Metrics** | Internal (Prometheus-like) | SLI/SLO tracking, dashboards |
| **Logging** | Scribe + Hive | Centralized log aggregation |
| **Tracing** | Distributed Tracing | Request flow across microservices |
| **Alerting** | Configerator | Config-driven, auto-generated alerts |
| **Chaos** | DR Storms + Fault Injection | Validate resilience proactively |

### 7.4 Incident Response

| Severity | Response Time | Example |
|---|---|---|
| **SEV-0** | < 5 min | Total site outage |
| **SEV-1** | < 15 min | Major feature degradation |
| **SEV-2** | < 1 hour | Partial outage, one region |
| **SEV-3** | < 4 hours | Non-critical service issue |
| **SEV-4** | Next business day | Minor bug, cosmetic issue |

---

## Mapping Tổng Hợp → NestJS

| Subsystem | Instagram | NestJS Implementation |
|---|---|---|
| **Media Pipeline** | DAG + Celery + CDN | `@nestjs/bull` DAG jobs + S3 + CloudFront |
| **Social Graph** | TAO (Objects + Assoc.) | Neo4j / TypeORM relations + Redis cache |
| **Data Pipeline** | Kafka → Flink → Hive | `@nestjs/microservices` Kafka → ClickHouse |
| **ML Ranking** | Two Towers + FAISS | TensorFlow Serving + `pgvector` |
| **Search** | Unicorn + Embeddings | Elasticsearch + `@nestjs/elasticsearch` |
| **Real-time** | MQTT + WebSocket | `@nestjs/websockets` + Socket.IO |
| **Notifications** | Kafka + APNs/FCM | `@nestjs/bull` + `firebase-admin` + `node-apn` |
| **Observability** | Scribe + Configerator | OpenTelemetry + Grafana + Prometheus |
| **DR** | DR Storms + FBAR | K8s PDB + Liveness probes + Chaos Mesh |
