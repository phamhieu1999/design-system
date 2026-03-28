# Netflix - Subsystems Analysis

> Recommendation, Search, Data Pipeline, A/B Testing, Observability.

---

## 1. Recommendation Engine

```mermaid
flowchart TB
    subgraph "Data Collection"
        D1["👁️ View history"]
        D2["⏱️ Watch time per title"]
        D3["🔍 Search queries"]
        D4["👍 Thumbs up/down"]
        D5["📋 My List additions"]
        D6["⏭️ Skip/abandon patterns"]
    end

    D1 & D2 & D3 & D4 & D5 & D6 --> FEATURES["Feature Store"]

    FEATURES --> MODELS["📊 Multiple ML Models"]

    subgraph "Model Types"
        CF["Collaborative Filtering<br/>'Users like you watched...'"]
        CB["Content-Based<br/>'Similar to what you watched...'"]
        SEQ["Sequential Model<br/>'Based on your last watch...'"]
        CONTEXT["Contextual<br/>'Time of day, device, mood'"]
    end

    MODELS --> CF & CB & SEQ & CONTEXT
    CF & CB & SEQ & CONTEXT --> RANK4["🏆 Ensemble Ranking"]
    RANK4 --> ROWS["📺 Homepage Rows<br/>'Because you watched X'<br/>'Trending Now'<br/>'Top Picks for You'"]

    style RANK4 fill:#e64980,color:#fff
```

### Personalized Everything

| Element | Personalization |
|---|---|
| **Homepage rows** | Row order, row content, row title |
| **Artwork** | Different poster per user (A/B tested!) |
| **Search results** | Ranked by personal relevance |
| **Autoplay preview** | Selected based on engagement prediction |
| **Continue watching** | Order based on likely resume |

---

## 2. A/B Testing — Netflix Culture

```mermaid
flowchart TB
    IDEA["💡 Feature idea"] --> HYPOTHESIS["📝 Hypothesis<br/>'Changing X will improve metric Y by Z%'"]
    HYPOTHESIS --> DESIGN["🔬 Experiment Design<br/>• Control group<br/>• Treatment group(s)<br/>• Sample size"]
    DESIGN --> DEPLOY["🚀 Deploy via Spinnaker<br/>(Feature flag toggle)"]
    DEPLOY --> RUN["📊 Run experiment<br/>(Days to weeks)"]
    RUN --> ANALYZE3["🤖 Statistical Analysis<br/>• Confidence interval<br/>• Multiple comparisons correction"]
    ANALYZE3 --> DECIDE{"Significant?"}
    DECIDE -->|"Yes, positive"| SHIP["✅ Ship to 100%"]
    DECIDE -->|"No / negative"| KILL["❌ Kill experiment"]

    NOTE7["Netflix runs ~250 A/B tests<br/>simultaneously at any time"]

    style NOTE7 fill:#4c6ef5,color:#fff
```

**Key insight:** Thậm chí **poster artwork** của mỗi phim cũng được A/B test — mỗi user có thể thấy poster khác nhau cho cùng 1 phim!

---

## 3. Search Architecture

```mermaid
flowchart TB
    QUERY2["🔍 User types 'stranger'"] --> INTENT["Intent Detection<br/>• Title? Actor? Genre?"]
    INTENT --> RETRIEVE["Elasticsearch Retrieval<br/>• Fuzzy match<br/>• Synonym expansion<br/>• Multi-language"]
    RETRIEVE --> PERSONALIZE["Personalize Ranking<br/>• User's taste profile<br/>• Watch history weight"]
    PERSONALIZE --> RESULTS3["📋 Results<br/>1. Stranger Things<br/>2. The Stranger<br/>3. Perfect Strangers"]

    subgraph "Search Features"
        TYPO["Typo tolerance: 'starnger' → 'Stranger'"]
        MULTI["Multi-language: title in 30+ languages"]
        VOICE["Voice search (mobile/TV)"]
    end
```

---

## 4. Data Pipeline

```mermaid
flowchart LR
    subgraph "Sources (Trillions events/day)"
        EV1["▶️ Play events"]
        EV2["👆 UI interactions"]
        EV3["📊 Quality metrics"]
        EV4["🖥️ Server logs"]
    end

    subgraph "Real-time Path"
        EV1 & EV2 & EV3 & EV4 --> KAFKA14["📨 Kafka"]
        KAFKA14 --> FLINK3["⚡ Flink<br/>(Real-time)"]
        FLINK3 --> KEYSTONE["🔑 Keystone<br/>(Netflix event pipeline)"]
    end

    subgraph "Batch Path"
        KAFKA14 --> S3_4["📦 S3 (Data Lake)"]
        S3_4 --> SPARK3["🔥 Spark<br/>(Batch analytics)"]
    end

    subgraph "Serving"
        FLINK3 --> DASH["📊 Real-time Dashboards"]
        SPARK3 --> MODELS2["🤖 ML Model Training"]
        SPARK3 --> REPORTS["📈 Business Reports"]
    end
```

---

## 5. Observability — Netflix Atlas

| Component | Tool | Purpose |
|---|---|---|
| **Metrics** | Atlas (Netflix internal) | Time-series, 2B+ data points/min |
| **Logging** | Edgar (Netflix internal) | Distributed request tracing |
| **Tracing** | Mantis (Netflix internal) | Real-time event processing |
| **Alerting** | PagerDuty + internal | SLO-based alerting |
| **Dashboards** | Lumen (Netflix internal) | Service health visualization |

---

## 6. So Sánh Tổng Hợp: 4 Systems

| Dimension | Netflix | Instagram | Twitter | WhatsApp |
|---|---|---|---|---|
| **Primary** | Video streaming | Photo/Video social | Microblogging | Messaging |
| **Language** | Java (Spring Boot) | Python (Django) | Scala/Java | Erlang/OTP |
| **Cloud** | AWS | Meta DCs | Google Cloud | Meta DCs |
| **CDN** | Open Connect (own) | Meta CDN | Generic CDN | N/A |
| **Search** | Elasticsearch | Unicorn | Earlybird | N/A |
| **ML** | Recommendation for everything | Explore/Feed ranking | Trending + ranking | Spam detection |
| **Unique** | Chaos Engineering | TAO social graph | Snowflake IDs | E2EE at scale |
| **OSS Contributions** | Eureka, Zuul, Spinnaker | Less public | Zipkin, Finagle | Less public |
| **Resilience** | Circuit breaker + Chaos | Redundancy | Priority queues | Supervision trees |
| **A/B Testing** | 250+ simultaneous tests | Extensive | Moderate | Minimal |

---

## Netflix Unique Innovations

| Innovation | Impact | Industry Adoption |
|---|---|---|
| **Netflix OSS** | Pioneered microservices tooling | Spring Cloud ecosystem |
| **Chaos Engineering** | "Test in production" culture | Adopted by Amazon, Google, Microsoft |
| **Open Connect** | CDN inside ISPs (free!) | Unique — no other company does this |
| **Per-title Encoding** | 20% bandwidth savings | Adopted by YouTube, Disney+ |
| **VMAF** | Perceptual quality metric | Open-source, industry standard |
| **Spinnaker** | Multi-region CD | CNCF project, used by Google, SAP |
| **Titus** | Container platform | Internal, but influenced K8s ecosystem |

---

## Mapping → NestJS

| Subsystem | Netflix | NestJS Implementation |
|---|---|---|
| **Recommendation** | Custom ML ensemble | TensorFlow.js / Python ML service via gRPC |
| **A/B Testing** | Internal platform | LaunchDarkly / Unleash + `@nestjs/config` |
| **Search** | Elasticsearch | `@nestjs/elasticsearch` |
| **Data Pipeline** | Kafka → Flink → S3 | `@nestjs/microservices` Kafka → ClickHouse |
| **Metrics** | Atlas | Prometheus + Grafana + OpenTelemetry |
| **Tracing** | Edgar | Jaeger / Zipkin + `nestjs-otel` |
| **Feature Flags** | Internal | Unleash / LaunchDarkly SDK |
