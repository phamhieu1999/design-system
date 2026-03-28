# Uber - Subsystems Analysis

> Maps, Uber Eats, Payments, Data Platform, ML (Michelangelo), Reliability.

---

## 1. Maps & Routing

```mermaid
flowchart TB
    subgraph "Uber Maps Platform"
        OSM["🗺️ OpenStreetMap (Base)"]
        PROBES["📡 Driver GPS probes<br/>(Billions of data points)"]
        THIRD["🌍 3rd-party data<br/>(Traffic, satellite)"]
    end

    OSM & PROBES & THIRD --> FUSE["🔀 Map Fusion Engine"]

    FUSE --> GRAPH["📊 Road Graph<br/>(Nodes + edges + weights)"]
    GRAPH --> ROUTING2["🗺️ Routing:<br/>• Dijkstra / A*<br/>• Real-time traffic weights<br/>• ML-enhanced predictions"]

    ROUTING2 --> ETA3["⏱️ ETA Prediction"]
    ROUTING2 --> NAV["🧭 Turn-by-turn Navigation"]
    ROUTING2 --> MATCH2["🚗 Map Matching<br/>(Snap GPS to road)"]

    style PROBES fill:#4c6ef5,color:#fff
```

**Key insight:** Uber uses **driver GPS probes** to build and continuously improve their own maps — billions of GPS points create highly accurate real-time traffic data.

---

## 2. Uber Eats Architecture

```mermaid
sequenceDiagram
    actor Customer as 👤 Customer
    participant App as Eats App
    participant API3 as API Gateway
    participant Order as Order Service
    participant Rest as Restaurant
    participant Match3 as Matching Service
    actor Courier as 🚴 Courier

    Customer->>App: Browse & order food
    App->>API3: Create order
    API3->>Order: Process order
    Order->>Rest: Send to restaurant POS

    Rest-->>Order: ✅ Accept + prep time (20min)

    Note over Order,Match3: At prep_time - 5min:
    Order->>Match3: Find courier
    Match3->>Match3: H3 nearby couriers<br/>+ ETA to restaurant<br/>+ ETA to customer (batched)
    Match3->>Courier: 🔔 Delivery offer

    Courier-->>Match3: ✅ Accept
    Courier->>Rest: 🚴 Go to restaurant
    Rest-->>Courier: 🍔 Pick up food
    Courier->>Customer: 🚴 Deliver
    Customer-->>Order: ✅ Delivered

    Note over Customer,Courier: Three-sided marketplace:<br/>Customer ↔ Restaurant ↔ Courier
```

### Eats-specific Challenges

| Challenge | Solution |
|---|---|
| **Prep time prediction** | ML model per restaurant |
| **Batched deliveries** | 2 orders to same area = 1 courier |
| **Cold food** | Match courier timing with prep completion |
| **Restaurant ranking** | Personalized by cuisine preference + distance |

---

## 3. Michelangelo — ML Platform

```mermaid
flowchart TB
    subgraph "End-to-End ML Pipeline"
        DATA["📊 Data Collection<br/>(Kafka → HDFS)"]
        FEATURE["🔧 Feature Store<br/>(Online: Redis, Offline: Hive)"]
        TRAIN["🏋️ Model Training<br/>(Spark, TensorFlow, XGBoost)"]
        EVAL["📏 Model Evaluation<br/>(A/B tests)"]
        DEPLOY3["🚀 Model Deployment<br/>(Online serving)"]
        MONITOR2["👁️ Model Monitoring<br/>(Drift detection)"]
    end

    DATA --> FEATURE --> TRAIN --> EVAL --> DEPLOY3 --> MONITOR2

    subgraph "ML Use Cases at Uber"
        U1["⏱️ ETA Prediction"]
        U2["💰 Surge Pricing"]
        U3["🤖 Fraud Detection"]
        U4["🍔 Restaurant Ranking"]
        U5["🚗 Driver/Rider Matching"]
        U6["📊 Demand Forecasting"]
        U7["🗺️ Map Quality"]
        U8["🛡️ Safety (RideCheck)"]
    end

    style FEATURE fill:#4c6ef5,color:#fff
```

---

## 4. Data Platform

```mermaid
flowchart LR
    subgraph "Sources"
        T1["Trips"] & T2["Payments"] & T3["GPS pings"] & T4["App events"]
    end

    subgraph "Ingestion"
        T1 & T2 & T3 & T4 --> KAFKA18["📨 Kafka<br/>(Trillions events/day)"]
    end

    subgraph "Real-time"
        KAFKA18 --> FLINK6["⚡ Flink"]
        FLINK6 --> PINOT2["📊 Apache Pinot<br/>(Real-time OLAP)"]
    end

    subgraph "Batch"
        KAFKA18 --> HDFS4["📦 HDFS / Data Lake"]
        HDFS4 --> SPARK5["🔥 Spark"]
        HDFS4 --> PRESTO4["🔍 Presto (SQL)"]
    end

    subgraph "Serving"
        PINOT2 --> DASH2["📊 Real-time Dashboards"]
        PRESTO4 --> ANALYTICS2["📈 Business Analytics"]
        SPARK5 --> ML7["🤖 ML Training"]
    end
```

**Apache Pinot** — Created by LinkedIn, heavily used by Uber for real-time analytics with sub-second query latency on massive datasets.

---

## 5. Reliability

```mermaid
flowchart TB
    subgraph "Patterns"
        CB4["🔌 Circuit Breaker<br/>(Hystrix-style)"]
        LB5["⚖️ Client-side Load Balancing"]
        RETRY3["🔄 Retry with backoff"]
        HEDGE["🏃 Hedged Requests<br/>(Send to 2 servers, use fastest)"]
        SHED2["🚫 Load Shedding<br/>(Drop low-priority requests)"]
    end

    subgraph "Graceful Degradation"
        DEG1["🟢 Normal: Full features"]
        DEG2["🟡 Degraded: Cached ETAs, no surge"]
        DEG3["🔴 Emergency: Basic matching only"]
    end

    subgraph "Observability (M3)"
        M3_METRICS["📊 M3 Metrics<br/>(Uber-built time-series DB)"]
        JAEGER["🔍 Jaeger<br/>(Distributed tracing, by Uber!)"]
    end

    style JAEGER fill:#4c6ef5,color:#fff
```

**Fun fact:** Uber created **Jaeger** — open-source distributed tracing (like Twitter's Zipkin).

---

## 6. So Sánh Tổng Hợp: 6 Systems

| Dimension | Uber | YouTube | Netflix | Instagram | Twitter | WhatsApp |
|---|---|---|---|---|---|---|
| **Primary** | Ride-hailing | Video UGC | Streaming | Photo social | Microblog | Messaging |
| **Language** | Go / Java | Python/C++/Go | Java (Spring) | Python | Scala/Java | Erlang |
| **Database** | MySQL+Cassandra | Vitess+Bigtable | Cassandra+Aurora | PostgreSQL | Manhattan | Mnesia+MySQL |
| **Unique tech** | H3, DOMA | Vitess, VP9 | Open Connect | TAO | Snowflake | BEAM hot swap |
| **ML Platform** | Michelangelo | TensorFlow internal | Internal | Internal | Internal | Minimal |
| **Tracing** | Jaeger (created) | Internal | Internal | Internal | Zipkin (created) | Internal |
| **Architecture** | DOMA | Google infra | Microservices | Monolith+services | JVM microservices | Erlang cluster |

---

## Uber Unique Innovations

| Innovation | Impact |
|---|---|
| **H3** | Open-source hex grid → used by Foursquare, Databricks, DoorDash |
| **DOMA** | Domain-oriented architecture → influenced microservice design |
| **Jaeger** | Open-source distributed tracing → CNCF project |
| **Michelangelo** | End-to-end ML platform → influenced MLOps industry |
| **Apache Pinot** | Real-time analytics → used by LinkedIn, Stripe, WePay |
| **Schemaless** | MySQL wrapper for schemaless data → influenced CockroachDB |

---

## Mapping → NestJS

| Subsystem | Uber | NestJS Implementation |
|---|---|---|
| **Maps/Routing** | GPS probes + graph routing | OSRM + `h3-js` + Redis |
| **Eats ordering** | Three-sided marketplace | `@nestjs/microservices` + state machine |
| **ML Platform** | Michelangelo | MLflow + Python gRPC service |
| **Real-time analytics** | Pinot | ClickHouse + Grafana |
| **Tracing** | Jaeger | `nestjs-otel` + Jaeger backend |
| **Hedged requests** | Dual-send, use fastest | Custom interceptor with `Promise.race` |
