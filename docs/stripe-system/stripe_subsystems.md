# Stripe - Subsystems Analysis

> Connect, Billing, Terminal, Atlas, Data Pipeline, Reliability.

---

## 1. Stripe Connect — Platform Payments

```mermaid
flowchart TB
    subgraph "Platform Model"
        PLATFORM["🏢 Platform<br/>(e.g., Shopify, Lyft)"]
        PLATFORM --> DIRECT["Direct charges<br/>(Platform is seller)"]
        PLATFORM --> DESTINATION["Destination charges<br/>(Seller receives, platform takes fee)"]
        PLATFORM --> TRANSFER["Separate charges + transfers"]
    end

    subgraph "Money Flow"
        CUSTOMER3["👤 Customer"] --> STRIPE2["Stripe"]
        STRIPE2 --> SPLIT2["Split:<br/>• Seller: $90<br/>• Platform fee: $7<br/>• Stripe fee: $3"]
        SPLIT2 --> SELLER["💰 Seller account"]
        SPLIT2 --> PLAT_ACCT["💰 Platform account"]
    end

    subgraph "Compliance"
        KYC["🔍 KYC (Know Your Customer)"]
        TAX["📋 1099 tax reporting"]
        ONBOARD["📝 Automated onboarding"]
    end
```

---

## 2. Stripe Billing — Subscription Engine

```mermaid
stateDiagram-v2
    [*] --> Active: Subscribe
    Active --> PastDue: Payment fails
    PastDue --> Active: Retry succeeds
    PastDue --> Canceled: Max retries exceeded
    Active --> Canceled: Customer cancels
    Active --> Trialing: Trial period
    Trialing --> Active: Trial ends
    Active --> Paused: Pause subscription
    Paused --> Active: Resume
    Canceled --> [*]
```

### Smart Retries (Revenue Recovery)

```mermaid
flowchart TB
    FAIL["❌ Payment fails"] --> SMART["🤖 Smart Retry"]

    subgraph "Smart Retry Logic"
        ML10["ML: Predict best retry time<br/>• Day of week<br/>• Time of day<br/>• Card type<br/>• Past retry success"]
    end

    SMART --> ML10
    ML10 --> RETRY6["🔄 Retry at optimal time"]
    RETRY6 -->|"Success"| RECOVER["✅ Revenue recovered!"]
    RETRY6 -->|"After N retries"| DUNNING["📧 Dunning emails<br/>→ Update card"]

    NOTE13["Stripe recovers 30-40% of<br/>initially failed payments"]

    style RECOVER fill:#51cf66,color:#fff
    style NOTE13 fill:#4c6ef5,color:#fff
```

---

## 3. Stripe Terminal — In-Person Payments

```mermaid
flowchart LR
    READER["📟 Card Reader<br/>(Stripe Terminal)"] --> SDK3["SDK<br/>(POS app)"]
    SDK3 --> STRIPE3["Stripe API<br/>(Same as online!)"]
    STRIPE3 --> PROCESS3["Same pipeline:<br/>• Tokenization<br/>• Radar<br/>• Ledger"]

    NOTE14["Unified online + in-person<br/>reporting & reconciliation"]
```

---

## 4. Financial Data Pipeline

```mermaid
flowchart LR
    subgraph "Events"
        E_PAY["Payments"]
        E_REFUND["Refunds"]
        E_DISPUTE["Disputes"]
        E_SUB["Subscriptions"]
    end

    E_PAY & E_REFUND & E_DISPUTE & E_SUB --> KAFKA20["📨 Kafka"]

    KAFKA20 --> LEDGER3["📒 Ledger Service<br/>(Double-entry, immutable)"]
    KAFKA20 --> ANALYTICS3["📊 Analytics<br/>(Sigma / Data Pipeline)"]
    KAFKA20 --> WEBHOOK2["📨 Webhook Delivery"]
    KAFKA20 --> RECONCILE["🔄 Reconciliation<br/>(Match with bank settlements)"]

    LEDGER3 --> REPORTING["📈 Revenue reporting<br/>+ Tax calculation"]
```

---

## 5. Reliability — 99.9995% Target

```mermaid
flowchart TB
    subgraph "Reliability Strategies"
        R1["📦 Cell-based isolation"]
        R2["🔄 Automatic failover"]
        R3["📊 Real-time anomaly detection"]
        R4["🐤 Canary deploys"]
        R5["⚡ Circuit breakers"]
        R6["💾 Read replicas for reporting"]
    end

    subgraph "Observability"
        O1["📊 Metrics (latency, error rate, throughput)"]
        O2["🔍 Distributed tracing"]
        O3["📋 Structured logging"]
        O4["🚨 Automated alerting + runbooks"]
    end
```

---

## 6. So Sánh Tổng Hợp: 8 Systems

| Dimension | Stripe | Amazon | Uber | YouTube | Netflix | Instagram | Twitter | WhatsApp |
|---|---|---|---|---|---|---|---|---|
| **Primary** | Payments API | E-commerce | Ride-hailing | Video UGC | Streaming | Photo social | Microblog | Messaging |
| **Language** | Ruby/Java/Go | Java/Kotlin | Go/Java | Python/C++ | Java | Python | Scala/Java | Erlang |
| **Key pattern** | Idempotency | Event-driven | Geospatial | Video pipeline | Chaos eng | Fan-out | Push/pull | Store-forward |
| **Database** | PostgreSQL | DynamoDB | MySQL+Cass | Vitess | Cassandra | PostgreSQL | Manhattan | Mnesia |
| **Unique** | Double-entry ledger | Bezos Mandate | H3 hex grid | Content ID | Open Connect | TAO graph | Snowflake | E2EE |
| **Availability** | 99.9995% | 99.99% | 99.99% | 99.99% | 99.97% | 99.95% | 99.95% | 99.99% |
| **Compliance** | PCI DSS L1 SP | PCI DSS L1 | PCI DSS | COPPA | GDPR | GDPR | GDPR | GDPR |

---

## Stripe Unique Innovations

| Innovation | Impact |
|---|---|
| **Idempotency keys** | Standard pattern for payment APIs → adopted industry-wide |
| **API versioning** | Date-based versioning with backward compat transforms |
| **Stripe Radar** | Network-wide ML fraud → $60B+ blocked/year |
| **Connect** | Multi-party payment orchestration for platforms |
| **Smart Retries** | ML-optimized retry timing → 30-40% recovery |
| **Developer Experience** | Set the gold standard for API documentation |

---

## Mapping → NestJS

| Subsystem | Stripe | NestJS Implementation |
|---|---|---|
| **Connect** | Multi-party payments | Stripe Connect SDK + custom splits |
| **Billing** | Subscription lifecycle | Stripe Billing API + webhook handlers |
| **Smart Retries** | ML retry timing | BullMQ delayed jobs + heuristic timing |
| **Terminal** | In-person payments | Stripe Terminal SDK |
| **Ledger** | Double-entry bookkeeping | PostgreSQL + 2 rows per txn |
| **Reconciliation** | Match bank settlements | Scheduled cron job + Stripe reporting API |
| **Observability** | Real-time monitoring | Prometheus + Grafana + `nestjs-otel` |
