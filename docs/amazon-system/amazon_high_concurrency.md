# Amazon - Xử Lý Đồng Thời Cao & E-Commerce

> 150M+ DynamoDB req/s vào Prime Day, 100K+ deploys/ngày, 1.6M+ packages/ngày.

---

## 1. Checkout Flow — Distributed Transaction

```mermaid
sequenceDiagram
    actor Customer as 👤 Customer
    participant Cart as Cart Service
    participant Pricing as Pricing Service
    participant Inventory as Inventory Service
    participant Order as Order Service
    participant Payment as Payment Service
    participant Fulfillment as Fulfillment Service

    Customer->>Cart: Checkout
    Cart->>Pricing: Calculate total<br/>(price, tax, shipping, discounts)
    Pricing-->>Cart: Final price

    Cart->>Inventory: Reserve items<br/>(soft lock, TTL 10min)
    Inventory-->>Cart: ✅ Reserved

    Cart->>Order: Create order
    Order->>Payment: Authorize payment
    Payment-->>Order: ✅ Authorized

    Order->>Order: Commit order (ACID)
    Order->>Fulfillment: 📨 Event: order.confirmed
    Order->>Customer: ✅ Order confirmed!

    Note over Fulfillment: Async fulfillment begins...
    Fulfillment->>Fulfillment: Select optimal FC
    Fulfillment->>Fulfillment: Pick → Pack → Ship
```

---

## 2. DynamoDB — Heart of Amazon

```mermaid
graph TB
    subgraph "DynamoDB Architecture"
        REQ6["Request"] --> ROUTER2["Request Router"]
        ROUTER2 --> PART1["Partition 1"]
        ROUTER2 --> PART2["Partition 2"]  
        ROUTER2 --> PARTN["Partition N"]

        PART1 --> REP1["Replica 1 (Leader)"]
        PART1 --> REP2["Replica 2"]
        PART1 --> REP3["Replica 3"]
    end

    subgraph "Key Features"
        K1["⚡ Single-digit ms latency"]
        K2["📈 Auto-scaling to any throughput"]
        K3["🌍 Global Tables (multi-region)"]
        K4["🔒 Encryption at rest"]
        K5["💰 On-demand or provisioned"]
    end

    style ROUTER2 fill:#e64980,color:#fff
```

### Hot Partition Prevention

```mermaid
flowchart TB
    PROBLEM["❌ Hot Partition:<br/>All requests to same key"] --> SOLUTION["✅ Write Sharding"]

    SOLUTION --> EXAMPLE["Key: product_id#[0-9]<br/>Scatter writes across 10 partitions"]
    EXAMPLE --> READ["Read: Query all 10 → aggregate"]

    subgraph "Design Patterns"
        DP1["1. High-cardinality partition keys"]
        DP2["2. Write sharding (suffix)"]
        DP3["3. Composite sort keys"]
        DP4["4. DAX (in-memory cache)"]
    end
```

---

## 3. Inventory Management — Chaotic Storage

```mermaid
flowchart TB
    subgraph "Fulfillment Center Layout"
        RECEIVE["📦 Receive"] --> STOW["📍 Chaotic Stow<br/>(Random location!)"]
        STOW --> PODS["🤖 Robot pods<br/>(Kiva/Amazon Robotics)"]
    end

    subgraph "Why Random?"
        W1["✅ Max space utilization"]
        W2["✅ Faster stowing"]
        W3["✅ Distribute pick paths"]
        W4["✅ System knows exact location"]
    end

    subgraph "Pick Flow"
        ORDER2["📋 Order received"] --> WMS["WMS: Find item location"]
        WMS --> ROBOT["🤖 Robot brings pod<br/>to picker station"]
        ROBOT --> PICK["👤 Human picks item"]
        PICK --> PACK3["📦 Auto-pack<br/>(Optimal box size)"]
        PACK3 --> SORT["🔀 Sort → Delivery station"]
    end

    style STOW fill:#e64980,color:#fff
```

---

## 4. Search & Product Discovery

```mermaid
flowchart TB
    QUERY3["🔍 'wireless headphones'"] --> TOKENIZE2["Tokenize + expand synonyms"]
    TOKENIZE2 --> RETRIEVE2["OpenSearch retrieval<br/>(Full-text + filters)"]
    RETRIEVE2 --> RANK5["🤖 ML Ranking:<br/>• Relevance<br/>• Conversion probability<br/>• Delivery speed<br/>• Seller quality<br/>• Sponsored (ads)"]
    RANK5 --> RESULTS4["📋 Results page"]

    subgraph "Special Features"
        AUTOCOMPLETE["Auto-complete (prefix index)"]
        VISUAL["📸 Visual search (image → products)"]
        VOICE2["🔊 Voice search (Alexa)"]
        RUFUS["🤖 Rufus AI assistant"]
    end
```

---

## 5. Prime Day — Handling 100x Traffic

```mermaid
flowchart TB
    PRIME["🎉 Prime Day<br/>(150M+ DDB req/s)"] --> PREP

    subgraph "Preparation (Weeks before)"
        PREP1["📊 Capacity planning<br/>(Historical + predicted)"]
        PREP2["🏋️ Load testing<br/>(GameDay exercises)"]
        PREP3["⚡ Pre-warm DynamoDB tables"]
        PREP4["📦 Pre-position inventory"]
        PREP5["🌍 CDN cache warm-up"]
    end

    PREP --> PREP1 & PREP2 & PREP3 & PREP4 & PREP5

    subgraph "During Prime Day"
        D1["⚡ Auto-scaling (all layers)"]
        D2["🔴 ElastiCache absorbs read load"]
        D3["📊 Real-time monitoring"]
        D4["🚫 Graceful degradation<br/>(Disable non-critical features)"]
        D5["🔌 Circuit breakers on every call"]
    end

    subgraph "Graceful Degradation Tiers"
        T1["🟢 Normal: Full personalization"]
        T2["🟡 Tier 1: Static recommendations"]
        T3["🟠 Tier 2: Cached product pages"]
        T4["🔴 Tier 3: Core checkout only"]
    end
```

---

## 6. Event-Driven Architecture

```mermaid
flowchart LR
    subgraph "Event Producers"
        P_A["Order placed"]
        P_B["Payment processed"]
        P_C["Item shipped"]
        P_D["Review submitted"]
    end

    P_A & P_B & P_C & P_D --> EB2["🔀 EventBridge"]

    EB2 --> C_A["📧 Notification Service<br/>(Email, push)"]
    EB2 --> C_B["📊 Analytics Pipeline"]
    EB2 --> C_C["🤖 Recommendation Update"]
    EB2 --> C_D["📦 Inventory Service"]
    EB2 --> C_E["💰 Seller Dashboard"]
```

---

## Mapping → NestJS

| Pattern | Amazon | NestJS Implementation |
|---|---|---|
| **Distributed checkout** | Saga pattern | `nestjs-saga` / custom orchestrator |
| **Inventory reserve** | Soft lock + TTL | Redis `SETNX` with TTL |
| **DynamoDB** | Auto-scaling NoSQL | `@aws-sdk/client-dynamodb` / MongoDB |
| **Write sharding** | Suffix partition key | Application-level sharding |
| **Chaotic storage** | WMS tracking | PostgreSQL + location table |
| **Event-driven** | EventBridge | `@nestjs/event-emitter` + Kafka |
| **Search** | OpenSearch | `@nestjs/elasticsearch` |
| **Caching** | ElastiCache | `@nestjs/cache-manager` + Redis |
