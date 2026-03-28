# Stripe - Xử Lý Đồng Thời Cao & Payment Processing

> Billions API calls/ngày, exactly-once payment semantics, < 100ms fraud scoring.

---

## 1. Payment Processing Flow

```mermaid
sequenceDiagram
    actor Customer as 👤 Customer
    participant Frontend as 🌐 Merchant Frontend
    participant Elements as 💳 Stripe Elements
    participant API4 as Stripe API
    participant Vault as 🔐 Token Vault
    participant Radar as 🤖 Radar (Fraud)
    participant Router as 🔀 Payment Router
    participant Network as 💳 Card Network
    participant Issuer as 🏦 Issuing Bank

    Customer->>Frontend: Enter card details
    Frontend->>Elements: Card data (never touches merchant!)
    Elements->>Vault: Tokenize card → tok_xxx
    Vault-->>Elements: Payment token

    Frontend->>API4: POST /v1/payment_intents/confirm<br/>+ Idempotency-Key: uuid-123

    API4->>API4: Check idempotency key
    API4->>Vault: Resolve token → encrypted PAN
    API4->>Radar: Score transaction (< 100ms)

    alt Radar: High risk
        Radar-->>API4: 🚫 Block
        API4-->>Frontend: requires_action (3D Secure)
    else Radar: Low risk
        Radar-->>API4: ✅ Allow
    end

    API4->>Router: Route to optimal network
    Router->>Network: Authorization request
    Network->>Issuer: Verify funds + approve
    Issuer-->>Network: ✅ Approved
    Network-->>Router: Auth code
    Router-->>API4: ✅ Authorized

    API4->>API4: Write to ledger (double-entry)
    API4-->>Frontend: ✅ Payment succeeded
    API4->>API4: Queue webhook event

    Note over API4: Later: Capture → Settlement → Payout
```

---

## 2. Idempotency — Exactly-Once Payments

```mermaid
flowchart TB
    REQ8["POST /charges<br/>Idempotency-Key: abc-123"] --> CHECK5{"Key exists in cache?"}

    CHECK5 -->|"No (first time)"| PROCESS2["Process payment"]
    PROCESS2 --> SAVE["Save result against key<br/>(TTL: 24 hours)"]
    SAVE --> RESPONSE["Return result"]

    CHECK5 -->|"Yes (retry)"| CACHED["Return cached result<br/>(Same response as first time)"]

    subgraph "Race Condition Prevention"
        LOCK2["🔒 Distributed lock on key"]
        LOCK2 --> ATOMIC["Atomic check-and-set<br/>(PostgreSQL UNIQUE constraint)"]
    end

    style CACHED fill:#51cf66,color:#fff
```

### Implementation Pattern

```
Idempotency Flow:
1. Client generates UUID → Idempotency-Key header
2. Server: SELECT FROM idempotency_keys WHERE key = ?
3. If found → return cached response
4. If not → INSERT key (with lock), process, store result
5. Key expires after 24 hours
```

---

## 3. Double-Entry Ledger

```mermaid
flowchart TB
    CHARGE["💳 Charge $100"] --> ENTRIES["Double-Entry:<br/>DEBIT: customer_balance $100<br/>CREDIT: merchant_pending $97<br/>CREDIT: stripe_fee $3"]

    CAPTURE["📦 Item shipped → Capture"] --> ENTRIES2["DEBIT: merchant_pending $97<br/>CREDIT: merchant_available $97"]

    PAYOUT2["💰 Payout"] --> ENTRIES3["DEBIT: merchant_available $97<br/>CREDIT: bank_transfer $97"]

    NOTE12["Every transaction ALWAYS balances<br/>Total debits == Total credits<br/>Immutable audit trail"]

    style NOTE12 fill:#4c6ef5,color:#fff
```

---

## 4. Webhook Delivery System

```mermaid
sequenceDiagram
    participant Stripe as Stripe
    participant Queue as Webhook Queue
    participant Endpoint as 🏢 Merchant Endpoint

    Stripe->>Queue: Event: payment_intent.succeeded
    
    loop Retry up to 3 days
        Queue->>Endpoint: POST /webhooks (signed payload)
        
        alt 2xx Response
            Endpoint-->>Queue: ✅ 200 OK
            Queue->>Queue: Mark delivered ✅
        else Timeout / 5xx
            Endpoint-->>Queue: ❌ 500 Error
            Queue->>Queue: Schedule retry<br/>(Exponential backoff:<br/>1min, 5min, 30min, 2h, 8h...)
        end
    end
```

### Webhook Best Practices

| Rule | Reason |
|---|---|
| **Verify signature** | Prevent spoofed webhooks |
| **Respond 200 fast** | Process async (queue internally) |
| **Handle duplicates** | Webhooks are at-least-once |
| **Fetch fresh state** | Don't rely on webhook data alone |

---

## 5. Saga Pattern — Distributed Transactions

```mermaid
flowchart TB
    ORDER4["🛒 Order"] --> STEP1["Step 1: Create PaymentIntent"]
    STEP1 --> STEP2["Step 2: Reserve inventory"]
    STEP2 --> STEP3["Step 3: Confirm payment"]
    STEP3 --> STEP4["Step 4: Create fulfillment"]

    STEP3 -->|"Payment fails"| COMP2["Compensate: Release inventory"]
    STEP2 -->|"Inventory unavailable"| COMP1["Compensate: Cancel PaymentIntent"]

    subgraph "Why Saga over 2PC?"
        S_WHY1["✅ No distributed locks"]
        S_WHY2["✅ Each step is local transaction"]
        S_WHY3["✅ Compensating actions for rollback"]
        S_WHY4["✅ Eventually consistent"]
    end

    style COMP1 fill:#ff6b6b,color:#fff
    style COMP2 fill:#ff6b6b,color:#fff
```

---

## 6. Smart Payment Routing

```mermaid
flowchart TB
    PAYMENT2["💳 Payment"] --> ANALYZE4["Analyze:<br/>• Card brand<br/>• Currency<br/>• Country<br/>• Amount<br/>• 3DS requirement"]

    ANALYZE4 --> ROUTE{"Route to optimal processor"}
    ROUTE -->|"US Visa"| PROC_A["Processor A (lowest fee)"]
    ROUTE -->|"EU Mastercard"| PROC_B["Processor B (local acquirer)"]
    ROUTE -->|"APAC"| PROC_C["Processor C (regional)"]

    PROC_A & PROC_B & PROC_C --> FAILOVER{"Success?"}
    FAILOVER -->|"Yes"| OK3["✅ Done"]
    FAILOVER -->|"Soft decline"| RETRY5["🔄 Retry via different processor"]

    style RETRY5 fill:#ffd43b,color:#333
```

**Adaptive Acceptance:** Stripe auto-retries soft declines through alternative processors → increases authorization rate by 2-5%.

---

## Mapping → NestJS

| Pattern | Stripe | NestJS Implementation |
|---|---|---|
| **Idempotency key** | UUID header + PostgreSQL | Custom `@Idempotent()` decorator + Redis |
| **Double-entry ledger** | Balanced transactions | 2 rows per transaction in PostgreSQL |
| **Webhook delivery** | Queue + retry 3 days | BullMQ + exponential backoff |
| **Webhook verification** | HMAC-SHA256 signature | `crypto.timingSafeEqual` |
| **Saga pattern** | Compensating transactions | `nestjs-saga` / custom orchestrator |
| **Smart routing** | ML-optimized | Rules engine + weighted selection |
| **Rate limiting** | Per-API-key | `@nestjs/throttler` + Redis |
