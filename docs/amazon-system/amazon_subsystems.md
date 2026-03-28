# Amazon - Subsystems Analysis

> Recommendation, Fulfillment, Alexa, Advertising, Data Pipeline, Reliability.

---

## 1. Recommendation Engine ("Customers who bought...")

```mermaid
flowchart TB
    DATA2["📊 User Signals"]
    DATA2 --> SIG_VIEW["Views"]
    DATA2 --> SIG_PURCHASE["Purchases"]
    DATA2 --> SIG_CART2["Add to cart"]
    DATA2 --> SIG_SEARCH["Search queries"]
    DATA2 --> SIG_RATING["Ratings & reviews"]

    SIG_VIEW & SIG_PURCHASE & SIG_CART2 & SIG_SEARCH & SIG_RATING --> MODELS3["📊 Multiple Models"]

    subgraph "Model Types"
        CF2_M["Collaborative Filtering<br/>'Customers like you bought...'"]
        II["Item-to-Item<br/>'Frequently bought together...'"]
        PERSONAL["Personalized ranking<br/>(Per-user, per-page)"]
        SESSION2["Session-based<br/>'Based on your browsing...'"]
    end

    MODELS3 --> CF2_M & II & PERSONAL & SESSION2
    CF2_M & II & PERSONAL & SESSION2 --> BLEND["🏆 Ensemble blend"]
    BLEND --> WIDGETS["📱 Recommendation widgets<br/>on every page"]

    style BLEND fill:#e64980,color:#fff
```

**Item-to-Item Collaborative Filtering** — Amazon's original breakthrough (1998 paper): precompute item pairs → fast lookup at serving time → scales to millions of products.

---

## 2. Fulfillment Network

```mermaid
graph TB
    subgraph "Multi-Echelon Network"
        VENDOR["🏭 Vendor / Supplier"]
        VENDOR --> IXD["📦 Inbound Cross-dock"]
        IXD --> AWD["📦 AWD<br/>(Bulk storage)"]
        AWD --> FC["🏢 Fulfillment Center<br/>(Pick, pack, ship)"]
        FC --> SC["📦 Sort Center"]
        SC --> DS["🚚 Delivery Station"]
        DS --> CUSTOMER2["🏠 Customer"]
    end

    subgraph "Fulfillment Types"
        FBA["FBA (Fulfilled by Amazon)<br/>Seller sends to Amazon FC"]
        FBM["FBM (Fulfilled by Merchant)<br/>Seller ships directly"]
        PRIME2["Prime Now / Fresh<br/>Micro-fulfillment, < 2 hours"]
    end

    style FC fill:#4c6ef5,color:#fff
    style DS fill:#51cf66,color:#fff
```

---

## 3. Amazon Advertising Platform

```mermaid
flowchart TB
    AD_REQ["📱 Product page load"] --> AUCTION["🏦 Real-time Ad Auction<br/>(< 50ms)"]

    AUCTION --> BID2["Sponsored bidders<br/>(CPC model)"]
    BID2 --> RANK6["Rank by:<br/>• Bid amount<br/>• Relevance score<br/>• Expected CTR<br/>• Product quality"]
    RANK6 --> DISPLAY["📢 Sponsored products shown"]
    DISPLAY --> TRACK2["📊 Impression/click tracking"]

    subgraph "Ad Types"
        SP["Sponsored Products<br/>(In search results)"]
        SB["Sponsored Brands<br/>(Top of page)"]
        SD["Sponsored Display<br/>(Retargeting)"]
        DSP["Amazon DSP<br/>(Programmatic, off-Amazon)"]
    end
```

---

## 4. Data Pipeline

```mermaid
flowchart LR
    subgraph "Sources"
        S_CLICK["Clicks"]
        S_ORDER["Orders"]
        S_SEARCH2["Searches"]
        S_INVENTORY["Inventory levels"]
    end

    subgraph "Ingestion"
        S_CLICK & S_ORDER & S_SEARCH2 & S_INVENTORY --> KINESIS2["🌊 Kinesis<br/>(Real-time)"]
    end

    subgraph "Processing"
        KINESIS2 --> LAMBDA3["⚡ Lambda (Transform)"]
        KINESIS2 --> EMR["🔥 EMR / Spark (Batch)"]
    end

    subgraph "Storage"
        LAMBDA3 --> S3_6["📦 S3 (Data Lake)"]
        EMR --> S3_6
    end

    subgraph "Analytics"
        S3_6 --> ATHENA["🔍 Athena (SQL on S3)"]
        S3_6 --> REDSHIFT["📊 Redshift (Warehouse)"]
        S3_6 --> SAGEMAKER["🤖 SageMaker (ML)"]
    end
```

---

## 5. Reliability — GameDays

```mermaid
flowchart TB
    subgraph "Amazon's Resilience Culture"
        GAMEDAY["🎮 GameDay Exercise"]
        GAMEDAY --> INJECT["Inject failures:<br/>• Kill instances<br/>• Add latency<br/>• Corrupt data<br/>• Region failure"]
        INJECT --> OBSERVE["📊 Observe system behavior"]
        OBSERVE --> FIX["🔧 Fix weaknesses"]
        FIX --> REPEAT["🔄 Repeat before Prime Day"]
    end

    subgraph "Patterns"
        CB5["🔌 Circuit Breakers everywhere"]
        CELL2["📦 Cell-based architecture<br/>(Blast radius isolation)"]
        RETRY4["🔄 Retry + exponential backoff"]
        SHUFFLE["🔀 Shuffle sharding"]
    end

    style GAMEDAY fill:#4c6ef5,color:#fff
```

**Cell-based architecture:** Amazon isolates failures by grouping customers into cells. If Cell A fails, Cell B/C/D unaffected → limits blast radius.

---

## 6. So Sánh Tổng Hợp: 7 Systems

| Dimension | Amazon | Uber | YouTube | Netflix | Instagram | Twitter | WhatsApp |
|---|---|---|---|---|---|---|---|
| **Primary** | E-commerce | Ride-hailing | Video UGC | Streaming | Photo social | Microblog | Messaging |
| **Language** | Java/Kotlin | Go/Java | Python/C++ | Java | Python | Scala/Java | Erlang |
| **Architecture** | SOA → µservices | DOMA | Google infra | Microservices | Monolith+svc | JVM µsvc | Erlang cluster |
| **Database** | DynamoDB+Aurora | MySQL+Cass | Vitess+Bigtable | Cass+Aurora | PostgreSQL | Manhattan | Mnesia+MySQL |
| **Key pattern** | Event-driven | Geospatial | Video pipeline | Chaos eng | Fan-out | Push/pull | Store-forward |
| **Open source** | AWS services | H3, Jaeger | Vitess, VP9 | Netflix OSS | Fewer | Zipkin | Fewer |
| **Unique** | Bezos Mandate | H3 hex grid | Content ID | Open Connect | TAO graph | Snowflake | E2EE at 2B |

---

## Amazon Unique Innovations

| Innovation | Impact |
|---|---|
| **Bezos API Mandate** | Forced SOA → birthed AWS ($90B+ revenue) |
| **Two-Pizza Teams** | Influenced org design at Google, Spotify, etc. |
| **DynamoDB** | Industry-standard serverless NoSQL |
| **Item-to-Item CF** | Original recommendation algorithm (1998) |
| **Chaotic Storage** | Counterintuitive but optimal warehouse design |
| **GameDays** | Proactive resilience testing → industry standard |
| **Cell-based Arch** | Blast radius isolation pattern |

---

## Mapping → NestJS

| Subsystem | Amazon | NestJS Implementation |
|---|---|---|
| **Recommendation** | Item-to-Item CF | Redis sorted sets + precomputed pairs |
| **Fulfillment** | Multi-echelon WMS | State machine + BullMQ + PostgreSQL |
| **Advertising** | Real-time auction | Redis + sorted set (bid ranking) |
| **Data pipeline** | Kinesis → Lambda | Kafka → `@nestjs/microservices` consumers |
| **Search** | OpenSearch | `@nestjs/elasticsearch` |
| **Cell architecture** | Blast radius isolation | K8s namespace per cell + traffic routing |
| **GameDays** | Chaos injection | `chaos-mesh` (K8s) / custom fault injector |
