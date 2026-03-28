# YouTube - Deployment & Architecture

> YouTube phục vụ **2.5B+ monthly users**, **800M+ videos**, **1B+ giờ xem/ngày**.

---

## 1. Quy Mô

| Metric | Giá trị |
|---|---|
| Monthly Active Users | 2.5B+ |
| Videos in catalog | 800M+ |
| Watch hours/day | 1B+ |
| Uploads/minute | 500+ giờ video |
| Bandwidth | ~11% internet traffic toàn cầu |
| Revenue (2024) | $36B+ (ads) |
| Employees | 3,000+ |

---

## 2. Technology Stack

```mermaid
graph TB
    subgraph "Client"
        WEB5["🌐 Web (Polymer → custom WC)"]
        IOS3["📱 iOS (Swift)"]
        AND2["📱 Android (Kotlin)"]
        TV2["📺 Smart TV / Console"]
    end

    subgraph "Edge & Network"
        MAGLEV["⚖️ Maglev<br/>(Software Load Balancer)"]
        QUIC["⚡ QUIC Protocol<br/>(HTTP/3)"]
        GCDN["🌍 Google Global CDN"]
    end

    subgraph "Backend"
        PY["🐍 Python<br/>(Business logic, rapid dev)"]
        CPP["⚡ C++<br/>(Video processing, perf-critical)"]
        JAVA2["☕ Java<br/>(Microservices)"]
        GO2["🦫 Go<br/>(High-concurrency services)"]
    end

    subgraph "Data Stores"
        VITESS["🗄️ Vitess + MySQL<br/>(Sharded relational)"]
        BT["📊 Bigtable<br/>(NoSQL, metadata)"]
        SPANNER2["🌐 Spanner<br/>(Globally consistent)"]
        GCS["📦 Google Cloud Storage<br/>(Video files)"]
    end

    subgraph "ML & Processing"
        TF["🧠 TensorFlow<br/>(Recommendation, moderation)"]
        BORG["☸️ Borg<br/>(Cluster management)"]
        PUBSUB["📨 Pub/Sub + Kafka<br/>(Event streaming)"]
    end

    WEB5 & IOS3 & AND2 & TV2 --> MAGLEV
    MAGLEV --> PY & JAVA2
    PY --> VITESS & BT
    CPP --> GCS
```

### Unique Google Infrastructure

| Component | Description |
|---|---|
| **Vitess** | MySQL sharding middleware — created by YouTube, now CNCF |
| **Bigtable** | Distributed NoSQL (wide-column) — billions of rows |
| **Spanner** | Globally consistent SQL — TrueTime atomic clocks |
| **Borg** | Cluster manager (predecessor to Kubernetes) |
| **Maglev** | Software load balancer — millions req/s per machine |
| **QUIC** | Transport protocol — faster than TCP (now HTTP/3) |
| **Colossus** | Distributed filesystem (successor to GFS) |

---

## 3. System Architecture

```mermaid
graph TB
    subgraph "Upload Path"
        CREATOR["🎬 Creator"] --> UPLOAD3["📤 Upload Service<br/>(Chunked, resumable)"]
        UPLOAD3 --> GCS2["📦 GCS (Raw file)"]
        GCS2 --> PIPELINE["🔀 Processing Pipeline<br/>(DAG scheduler)"]
        PIPELINE --> TRANSCODE3["⚡ Transcode Farm<br/>(1000s workers)"]
        TRANSCODE3 --> GCS3["📦 GCS (Processed)"]
        GCS3 --> CDN4["🌍 CDN Distribution"]
    end

    subgraph "Watch Path"
        VIEWER["👁️ Viewer"] --> MAGLEV2["⚖️ Maglev"]
        MAGLEV2 --> API2["API Server"]

        API2 --> AUTH3["🔑 Auth"]
        API2 --> REC["🤖 Recommendation"]
        API2 --> META["📋 Metadata (Vitess)"]

        API2 --> STEERING2["🧭 CDN Steering<br/>(Nearest edge)"]
        STEERING2 --> CDN4
        CDN4 --> VIEWER
    end
```

---

## 4. Vitess — MySQL Sharding at YouTube Scale

```mermaid
graph TB
    APP3["Application"] --> VTGATE["vtgate<br/>(Query router)"]

    VTGATE --> SHARD_1["Shard 1<br/>(MySQL)"]
    VTGATE --> SHARD_2["Shard 2<br/>(MySQL)"]
    VTGATE --> SHARD_N["Shard N<br/>(MySQL)"]

    SHARD_1 --> VTTAB1["vttablet<br/>(Connection pooling<br/>+ query rewriting)"]
    SHARD_2 --> VTTAB2["vttablet"]
    SHARD_N --> VTTABN["vttablet"]

    subgraph "Vitess Features"
        F_A["Transparent sharding"]
        F_B["Online schema changes"]
        F_C["Connection pooling"]
        F_D["Query rewriting & protection"]
        F_E["Automated resharding"]
    end

    style VTGATE fill:#4c6ef5,color:#fff
```

**Why Vitess?** YouTube needed MySQL's reliability but at Google scale (billions of rows). Vitess adds sharding, connection pooling, and online DDL without changing app code.

---

## 5. Deployment

```mermaid
flowchart TB
    CODE2["👨‍💻 Code"] --> BUILD4["🔨 Build (Blaze/Bazel)"]
    BUILD4 --> TEST4["🧪 Automated Tests"]
    TEST4 --> CANARY4["🐤 Canary (Google-scale)"]
    CANARY4 --> GRADUAL["📈 Gradual rollout<br/>(Cell by cell, region by region)"]
    GRADUAL --> FULL["✅ Full production"]

    subgraph "Infrastructure"
        BORG2["☸️ Borg<br/>(Google's cluster manager)"]
        NOTE8["Borg inspired Kubernetes<br/>but YouTube runs on Borg"]
    end

    style BORG2 fill:#4c6ef5,color:#fff
```

---

## Mapping → NestJS

| YouTube | NestJS Implementation |
|---|---|
| **Vitess + MySQL** | TypeORM + PostgreSQL (sharded) |
| **Bigtable** | Cassandra / ScyllaDB |
| **QUIC/HTTP3** | Node.js HTTP/2 + Caddy proxy |
| **Maglev** | Nginx / HAProxy / K8s Ingress |
| **Borg** | Kubernetes (K8s was inspired by Borg) |
| **Pub/Sub** | `@nestjs/microservices` Kafka |
| **TensorFlow** | TensorFlow.js / Python ML via gRPC |
