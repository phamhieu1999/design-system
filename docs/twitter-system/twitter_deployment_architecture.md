# Hệ Thống Twitter/X - Deployment & Architecture

> Twitter xử lý **500K+ tweets/phút**, **200K+ requests/giây** phục vụ 500M+ users.

---

## 1. Tổng Quan & Quy Mô

| Metric | Giá trị |
|---|---|
| Monthly Active Users | 500M+ |
| Tweets per day | 500M+ |
| Timeline views/day | 200B+ |
| Peak tweets/second | 150K+ (Super Bowl, World Cup) |
| Servers | Tens of thousands |
| Requests/second | 200K+ |

---

## 2. Technology Stack

```mermaid
graph TB
    subgraph "Frontend"
        WEB["🌐 React.js + TypeScript"]
        IOS["📱 Swift (iOS)"]
        ANDROID["📱 Kotlin (Android)"]
    end

    subgraph "API & RPC"
        GW2["🚪 API Gateway"]
        FINAGLE["⚡ Finagle<br/>(Scala RPC Framework)"]
    end

    subgraph "Backend Services (JVM)"
        SCALA["Scala / Java<br/>(Core Services)"]
        GO["Go<br/>(High-concurrency services)"]
        PYTHON["Python<br/>(ML/AI, Grok)"]
    end

    subgraph "Data Stores"
        MANHATTAN["🏙️ Manhattan<br/>(Distributed KV Store)"]
        REDIS2["🔴 Redis Cluster<br/>(Timeline Cache)"]
        MYSQL2["🐬 MySQL<br/>(Legacy, Ads)"]
        HDFS2["📊 Hadoop/HDFS<br/>(Batch Analytics)"]
        ES2["🔍 Elasticsearch<br/>(Users, DMs)"]
        EARLY["🐦 Earlybird<br/>(Real-time Tweet Search)"]
    end

    subgraph "Infrastructure"
        K8S["☸️ Kubernetes"]
        KAFKA6["📨 Apache Kafka"]
        GCP["☁️ Google Cloud"]
    end

    WEB & IOS & ANDROID --> GW2
    GW2 --> FINAGLE
    FINAGLE --> SCALA & GO & PYTHON
    SCALA --> MANHATTAN & REDIS2 & MYSQL2 & ES2 & EARLY
    SCALA --> KAFKA6
```

### Tech Stack Evolution

```mermaid
timeline
    title Twitter Tech Stack Evolution
    2006 : Ruby on Rails monolith
         : MySQL, Memcached
    2010 : "Fail Whale" era → migrate to JVM
         : Scala + Finagle framework
    2012 : Manhattan KV store
         : Snowflake ID generator
    2014 : Elasticsearch integration
         : Mesos + Aurora orchestration
    2019 : Kubernetes migration begins
    2022 : Google Cloud migration
         : Massive infra reduction (Elon era)
    2024 : AI/ML heavy (Grok)
         : Continued K8s + GCP
```

---

## 3. System Architecture - Tổng Quan

```mermaid
graph TB
    subgraph "Client Layer"
        C1["📱 Mobile Apps"]
        C2["🌐 Web Client"]
        C3["🤖 API Consumers"]
    end

    subgraph "Edge"
        CDN3["🌍 CDN"]
        LB2["⚖️ Load Balancer"]
    end

    subgraph "Core Services"
        TWEET_SVC["📝 Tweet Service"]
        TIMELINE_SVC["📰 Timeline Service"]
        USER_SVC["👤 User Service"]
        SEARCH_SVC["🔍 Search Service"]
        NOTIF_SVC["🔔 Notification Service"]
        DM_SVC["💬 DM Service"]
        MEDIA_SVC["📸 Media Service"]
        ADS_SVC["💰 Ads Service"]
        RECOMMEND["🤖 Recommendation (Grok)"]
    end

    subgraph "Fan-out Service"
        FANOUT2["📤 Fan-out Service<br/>Hybrid Push/Pull"]
    end

    subgraph "Storage"
        MH["🏙️ Manhattan"]
        RD["🔴 Redis Timeline Cache"]
        EB["🐦 Earlybird"]
        BLOB2["📦 Blob Storage"]
    end

    subgraph "Event Bus"
        KF["📨 Kafka"]
    end

    C1 & C2 & C3 --> CDN3 --> LB2
    LB2 --> TWEET_SVC & TIMELINE_SVC & USER_SVC & SEARCH_SVC & NOTIF_SVC & DM_SVC & MEDIA_SVC & ADS_SVC

    TWEET_SVC --> KF
    KF --> FANOUT2
    FANOUT2 --> RD
    TIMELINE_SVC --> RD & MH
    SEARCH_SVC --> EB
    MEDIA_SVC --> BLOB2
    RECOMMEND --> TIMELINE_SVC
```

---

## 4. Snowflake ID — Distributed ID Generation

Twitter tạo ra **Snowflake** để sinh unique IDs mà KHÔNG cần central coordinator.

```mermaid
graph LR
    subgraph "64-bit Snowflake ID"
        B0["0<br/>Sign<br/>1 bit"]
        B1["Timestamp<br/>(ms since epoch)<br/>41 bits"]
        B2["DC ID<br/>5 bits"]
        B3["Worker ID<br/>5 bits"]
        B4["Sequence<br/>12 bits"]
    end
    B0 --- B1 --- B2 --- B3 --- B4
```

| Component | Bits | Range |
|---|---|---|
| Sign | 1 | Always 0 |
| Timestamp | 41 | ~69 years từ custom epoch |
| Datacenter ID | 5 | 32 datacenters |
| Worker ID | 5 | 32 workers per DC |
| Sequence | 12 | 4,096 IDs/ms/worker |

**Throughput:** ~4 triệu IDs/giây/worker. Time-ordered → sortable by creation time.

---

## 5. Deployment & Infrastructure

```mermaid
flowchart TB
    subgraph "CI/CD Pipeline"
        COMMIT["👨‍💻 Commit"] --> BUILD["🔨 Build (Bazel)"]
        BUILD --> TEST["🧪 Tests"]
        TEST --> ARTIFACT["📦 Container Image"]
        ARTIFACT --> CANARY2["🐤 Canary (1-5%)"]
        CANARY2 --> STAGED["📊 Staged Rollout"]
        STAGED --> PROD["🚀 Production"]
    end

    subgraph "Infrastructure Evolution"
        BARE["🖥️ Bare Metal<br/>(Own DCs)"]
        MESOS["⚙️ Mesos + Aurora"]
        K8S2["☸️ Kubernetes"]
        GCP2["☁️ Google Cloud"]
        BARE --> MESOS --> K8S2 --> GCP2
    end

    style GCP2 fill:#4c6ef5,color:#fff
```

| Era | Infrastructure | Deployment |
|---|---|---|
| 2010-2014 | Bare metal, own DCs | Custom scripts, SSH |
| 2014-2019 | Mesos + Aurora | Aurora scheduler |
| 2019-2022 | Kubernetes migration | K8s declarative (YAML) |
| 2022+ | Google Cloud + K8s | GKE, standardized CI/CD |

---

## Mapping → NestJS

| Twitter/X | NestJS Implementation |
|---|---|
| **Scala + Finagle** | NestJS + gRPC (`@grpc/grpc-js`) |
| **Manhattan** | Redis + TypeORM (PostgreSQL) |
| **Snowflake ID** | `snowflake-id` / custom BigInt generator |
| **Kubernetes** | Docker + K8s (Helm charts) |
| **Bazel build** | `npm run build` + Docker multi-stage |
