# Stripe - Deployment & Architecture

> Stripe xử lý **hàng trăm tỷ USD/năm**, 99.9995% uptime — gold standard API-first payments.

---

## 1. Quy Mô

| Metric | Giá trị |
|---|---|
| Transaction volume | $1T+/năm |
| Businesses using Stripe | Millions |
| Countries supported | 46+ (direct), 135+ (global payments) |
| Availability target | 99.9995% ("five-and-a-half nines") |
| API requests/day | Billions |
| Fraud blocked (Radar) | $60B+/năm |

---

## 2. Technology Stack

```mermaid
graph TB
    subgraph "Client SDKs"
        ELEMENTS["💳 Stripe Elements<br/>(Client-side tokenization)"]
        CHECKOUT["🛒 Checkout<br/>(Hosted payment page)"]
        MOBILE4["📱 Mobile SDKs"]
    end

    subgraph "API Layer"
        API_V["🔗 Versioned REST API<br/>(api.stripe.com)"]
    end

    subgraph "Backend"
        RUBY["💎 Ruby<br/>(Core API, business logic)"]
        JAVA5["☕ Java<br/>(High-perf services)"]
        GO4["🦫 Go<br/>(Infrastructure services)"]
    end

    subgraph "Data Stores"
        PG["🐘 PostgreSQL<br/>(Primary, sharded)"]
        REDIS6["🔴 Redis<br/>(Cache, rate limiting)"]
        KAFKA19["📨 Kafka<br/>(Event streaming)"]
    end

    subgraph "Security"
        HSM2["🔐 HSM<br/>(Key management)"]
        VAULT2["🗝️ Vault<br/>(Tokenization)"]
    end

    subgraph "Infrastructure"
        AWS2["☁️ AWS (Primary)"]
        CELL3["📦 Cell-based Architecture"]
    end

    ELEMENTS & CHECKOUT & MOBILE4 --> API_V
    API_V --> RUBY
    RUBY --> PG & REDIS6 & KAFKA19
```

---

## 3. System Architecture

```mermaid
graph TB
    subgraph "Payment Flow"
        MERCHANT["🏢 Merchant App"] --> STRIPE_API["Stripe API"]
        STRIPE_API --> IDEM["🔑 Idempotency Check"]
        IDEM --> TOKEN5["🔐 Tokenization Vault"]
        TOKEN5 --> FRAUD2["🤖 Radar (Fraud)"]
        FRAUD2 --> ROUTING["🔀 Payment Router"]
    end

    subgraph "Card Network"
        ROUTING --> VISA["💳 Visa"]
        ROUTING --> MC["💳 Mastercard"]
        ROUTING --> AMEX["💳 Amex"]
        ROUTING --> LOCAL["💳 Local methods<br/>(iDEAL, SEPA, PIX)"]
    end

    subgraph "Post-Processing"
        VISA & MC & AMEX --> LEDGER2["📒 Double-Entry Ledger"]
        LEDGER2 --> PAYOUT["💰 Payout to merchant"]
        LEDGER2 --> WEBHOOK["📨 Webhook delivery"]
    end

    style IDEM fill:#4c6ef5,color:#fff
    style LEDGER2 fill:#51cf66,color:#fff
```

---

## 4. API Design — Industry Benchmark

```mermaid
flowchart TB
    subgraph "API Design Principles"
        V1["📋 Versioned APIs<br/>(2024-12-18 format)"]
        V2["🔑 Idempotency keys<br/>(Prevent double charges)"]
        V3["📄 Consistent object model<br/>(Expandable resources)"]
        V4["⚠️ Predictable error format<br/>(type, code, message)"]
        V5["🔄 Pagination<br/>(Cursor-based)"]
        V6["📨 Webhooks<br/>(Event-driven integration)"]
    end
```

### API Versioning Strategy

```mermaid
flowchart LR
    REQ7["API Request"] --> HEADER{"Stripe-Version header?"}
    HEADER -->|"Set"| SPECIFIC["Use specified version"]
    HEADER -->|"Not set"| DEFAULT3["Use account's pinned version"]

    subgraph "Version Compatibility"
        OLD["Old version:<br/>Stripe transforms response<br/>to match expected format"]
        NEW2["New version:<br/>Latest behavior"]
    end
```

**Key insight:** Stripe maintains backward compatibility by transforming API responses through version-specific mappers — merchants NEVER break on upgrade.

---

## 5. Cell-Based Architecture

```mermaid
graph TB
    subgraph "Cell Architecture"
        LB6["⚖️ Load Balancer"]
        LB6 --> CELL_A["📦 Cell A<br/>(Merchants 1-1000)"]
        LB6 --> CELL_B["📦 Cell B<br/>(Merchants 1001-2000)"]
        LB6 --> CELL_C["📦 Cell C<br/>(Merchants 2001-3000)"]

        CELL_A --> DB_A["DB Shard A"]
        CELL_B --> DB_B["DB Shard B"]
        CELL_C --> DB_C["DB Shard C"]
    end

    NOTE11["Cell A failure →<br/>ONLY Merchants 1-1000 affected<br/>Cells B, C unaffected"]

    style NOTE11 fill:#51cf66,color:#fff
```

---

## 6. Deployment

```mermaid
flowchart TB
    CODE5["👨‍💻 Code"] --> CI3["🧪 CI (Tests + type check)"]
    CI3 --> CANARY7["🐤 Canary (shadow traffic)"]
    CANARY7 --> GRADUAL2["📈 Gradual rollout"]
    GRADUAL2 --> PROD4["✅ Production"]

    subgraph "Safety"
        SAFE1["Zero-downtime deploys"]
        SAFE2["Automated rollback on error spike"]
        SAFE3["Feature flags for new functionality"]
    end
```

---

## Mapping → NestJS

| Stripe | NestJS Implementation |
|---|---|
| **API versioning** | Custom `@ApiVersion()` decorator + interceptor |
| **Idempotency key** | Custom middleware + Redis `SETNX` |
| **Cell architecture** | K8s namespace + DB connection per cell |
| **Double-entry ledger** | PostgreSQL + debit/credit rows |
| **Webhooks** | Bull queue + retry with exponential backoff |
| **Ruby core** | NestJS (TypeScript) |
| **HSM** | AWS KMS / HashiCorp Vault |
