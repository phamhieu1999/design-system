# Google Search - Deployment & Architecture

> Google xử lý **8.5B+ searches/ngày**, index **hundreds of billions** web pages, < 200ms response.

---

## 1. Quy Mô

| Metric | Giá trị |
|---|---|
| Searches/day | 8.5B+ (~100K/second) |
| Indexed pages | Hundreds of billions |
| Crawled pages/day | Billions |
| Response time | < 200ms (usually < 500ms) |
| Data centers | 30+ globally |
| Revenue (Ads) | $300B+/year |

---

## 2. Technology Stack

```mermaid
graph TB
    subgraph "Client"
        BROWSER2["🌐 Browser"]
        APP2["📱 App"]
        VOICE4["🔊 Voice (Assistant)"]
    end

    subgraph "Edge"
        DNS3["🌍 Google Global DNS"]
        GFE["⚖️ Google Front End (GFE)"]
    end

    subgraph "Core Infrastructure"
        CPP["🔧 C++ (Serving, performance-critical)"]
        JAVA7["☕ Java (Backend services)"]
        PY4["🐍 Python (ML pipelines)"]
        GO5["🦫 Go (Infrastructure tooling)"]
    end

    subgraph "Storage Systems"
        COLOSSUS["📦 Colossus (Distributed FS)"]
        BIGTABLE3["📀 Bigtable (NoSQL)"]
        SPANNER3["🗄️ Spanner (Global SQL)"]
    end

    subgraph "Compute"
        BORG2["☸️ Borg (Cluster manager)"]
        MAPREDUCE["🔀 MapReduce / Flume"]
        TENSOR["🧠 TPU (ML inference)"]
    end

    BROWSER2 & APP2 --> DNS3 --> GFE
    GFE --> CPP
    CPP --> COLOSSUS & BIGTABLE3
```

---

## 3. System Architecture Overview

```mermaid
flowchart TB
    subgraph "1. Crawl"
        GOOGLEBOT2["🕷️ Googlebot<br/>(Distributed crawler)"]
        FRONTIER["📋 URL Frontier<br/>(Priority queue)"]
        FETCH["📥 Fetch pages"]
    end

    subgraph "2. Process"
        PARSE["🔧 Parse HTML"]
        DEDUP["🔍 Deduplication<br/>(SimHash)"]
        EXTRACT["📝 Extract text, links,<br/>structured data"]
    end

    subgraph "3. Index"
        INVERTED["📇 Inverted Index<br/>(Term → DocIDs)"]
        KNOWLEDGE["🧠 Knowledge Graph"]
        LINKGRAPH["🔗 Link Graph<br/>(PageRank)"]
    end

    subgraph "4. Serve"
        QUERY5["🔍 Query Processing"]
        RETRIEVE3["📊 Retrieval"]
        RANK7["🤖 Multi-stage Ranking"]
        RESULTS6["📋 Search Results"]
    end

    GOOGLEBOT2 --> FRONTIER --> FETCH
    FETCH --> PARSE --> DEDUP --> EXTRACT
    EXTRACT --> INVERTED & KNOWLEDGE & LINKGRAPH
    INVERTED & KNOWLEDGE & LINKGRAPH --> QUERY5
    QUERY5 --> RETRIEVE3 --> RANK7 --> RESULTS6

    style INVERTED fill:#4285F4,color:#fff
    style RANK7 fill:#EA4335,color:#fff
```

---

## 4. Foundational Infrastructure

```mermaid
graph TB
    subgraph "Storage Layer"
        COLOSSUS2["📦 Colossus<br/>(Successor to GFS)<br/>Exabytes of data"]
        BIGTABLE4["📀 Bigtable<br/>(Wide-column NoSQL)<br/>Petabytes, ms latency"]
        SPANNER4["🗄️ Spanner<br/>(Global relational DB)<br/>ACID + TrueTime"]
    end

    subgraph "Compute Layer"
        BORG3["☸️ Borg → Kubernetes<br/>(Cluster management)"]
        MR["🔀 MapReduce<br/>(Batch processing)"]
        DREMEL["📊 Dremel → BigQuery<br/>(Interactive analytics)"]
    end

    subgraph "Network"
        JUPITER["🌐 Jupiter<br/>(Datacenter fabric)"]
        B4["🌍 B4<br/>(WAN backbone, SDN)"]
        MAGLEV2["⚖️ Maglev<br/>(Network load balancer)"]
    end
```

### TrueTime — Spanner's Secret Weapon

```
TrueTime API:
  tt.now()  → [earliest, latest]  (bounded uncertainty)
  
Uses atomic clocks + GPS receivers in every datacenter
→ Global clock sync within ~7ms uncertainty
→ Enables globally consistent transactions WITHOUT distributed locks
```

---

## 5. Deployment Scale

| Component | Scale |
|---|---|
| **Borg jobs** | Millions of tasks across clusters |
| **GFE (Frontend)** | Handles billions of requests/day |
| **Index shards** | Thousands of shards, replicated globally |
| **Colossus** | Exabytes of storage |
| **TPUs** | Thousands for ML inference |
| **B4 backbone** | Petabits/s capacity |

---

## Mapping → NestJS

| Google | NestJS Implementation |
|---|---|
| **Colossus/GFS** | S3 / MinIO (object storage) |
| **Bigtable** | Cassandra / ScyllaDB |
| **Spanner** | CockroachDB / PostgreSQL |
| **Borg** | Kubernetes (GKE/EKS) |
| **MapReduce** | Kafka + BullMQ workers |
| **GFE** | Nginx + `@nestjs/platform-express` |
| **TPU inference** | TensorFlow Serving via gRPC |
