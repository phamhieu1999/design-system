# Amazon - Deployment & Architecture

> Amazon phục vụ **310M+ customers**, **12M+ sản phẩm**, **1.6M+ packages/ngày** — pioneer SOA & cloud.

---

## 1. Quy Mô

| Metric | Giá trị |
|---|---|
| Active customers | 310M+ |
| Products | 12M+ (Amazon only), 350M+ (marketplace) |
| Packages/day | 1.6M+ |
| Prime Day peak | 150M+ DynamoDB req/s |
| Fulfillment centers | 175+ globally |
| AWS revenue | $90B+ (2024) |
| Deployments/day | 100,000+ (across all services) |

---

## 2. Technology Stack

```mermaid
graph TB
    subgraph "Client"
        WEB7["🌐 Web (React)"]
        IOS4["📱 iOS"]
        AND3["📱 Android"]
        ALEXA["🔊 Alexa"]
    end

    subgraph "Edge"
        CF["🌍 CloudFront CDN"]
        ALB2["⚖️ ALB (Load Balancer)"]
    end

    subgraph "Backend Services"
        JAVA4["☕ Java / Kotlin<br/>(Core services)"]
        PY2["🐍 Python<br/>(ML, data pipelines)"]
        RUST["🦀 Rust<br/>(Performance-critical)"]
    end

    subgraph "Data Stores"
        DDB["⚡ DynamoDB<br/>(NoSQL, core)"]
        AURORA2["🐦 Aurora<br/>(Relational)"]
        S3_5["📦 S3<br/>(Object storage)"]
        ELASTICACHE["🔴 ElastiCache<br/>(Redis/Memcached)"]
        OPENSEARCH["🔍 OpenSearch<br/>(Search)"]
    end

    subgraph "Messaging"
        SQS["📨 SQS (Queue)"]
        SNS["📢 SNS (Pub/Sub)"]
        KINESIS["🌊 Kinesis<br/>(Real-time streaming)"]
        EB["🔀 EventBridge<br/>(Event bus)"]
    end

    subgraph "Infrastructure"
        ECS["📦 ECS / EKS<br/>(Containers)"]
        LAMBDA2["⚡ Lambda<br/>(Serverless)"]
        CODEPIPELINE["🚀 CodePipeline + CodeDeploy"]
    end

    WEB7 & IOS4 & AND3 --> CF --> ALB2
    ALB2 --> JAVA4
    JAVA4 --> DDB & AURORA2 & ELASTICACHE
    JAVA4 --> SQS & SNS
```

---

## 3. Architecture Evolution

```mermaid
timeline
    title Amazon Architecture Evolution
    2000 : Monolith (C++ & Perl)
         : Single Oracle database
    2002 : "Bezos API Mandate"
         : All teams MUST expose APIs
    2004 : SOA migration
         : 2-pizza teams formed
    2006 : AWS launched (S3, EC2)
         : "Eat your own dog food"
    2010 : DynamoDB created
         : Microservices at scale
    2015 : 100K+ deploys/day
         : Aurora, Lambda launched
    2020 : Event-driven architecture
         : Step Functions orchestration
    2024 : AI everywhere (Rufus, Bedrock)
         : Graviton (ARM) adoption
```

### The Bezos API Mandate (2002)

> "All teams will henceforth expose their data and functionality through service interfaces. Teams must communicate through these interfaces. There will be no other form of interprocess communication. Anyone who doesn't do this will be fired." — Jeff Bezos

**Impact:** Biến Amazon từ monolith thành SOA → trở thành nền tảng cho AWS.

---

## 4. Two-Pizza Team Organization

```mermaid
graph TB
    subgraph "Team Structure"
        TEAM["🍕🍕 Two-Pizza Team<br/>(6-10 people)"]
        TEAM --> OWN["Own 1-2 services end-to-end"]
        OWN --> BUILD["Build"]
        OWN --> RUN["Run (you build it, you run it)"]
        OWN --> ONCALL["On-call"]
    end

    subgraph "Example: Cart Service Team"
        CART_TEAM["Cart Team"]
        CART_TEAM --> CART_SVC["Cart Service"]
        CART_TEAM --> CART_DB["Cart DynamoDB table"]
        CART_TEAM --> CART_API["Cart API"]
        CART_TEAM --> CART_MONITOR["Monitoring & alerts"]
    end
```

---

## 5. Deployment — 100K+ Deploys/Day

```mermaid
flowchart TB
    CODE4["👨‍💻 Code"] --> PR["Pull Request"]
    PR --> REVIEW["Code Review"]
    REVIEW --> BUILD5["🔨 Build + Unit Tests"]
    BUILD5 --> PIPELINE["🚀 CodePipeline"]
    
    PIPELINE --> GAMMA["🧪 Gamma (Pre-prod)<br/>(Full integration tests)"]
    GAMMA --> ONEBOX["📦 OneBox Deploy<br/>(1 server, real traffic)"]
    ONEBOX --> CANARY6["🐤 Canary (5% traffic)"]
    CANARY6 --> LINEAR["📈 Linear rollout<br/>(Region by region)"]
    LINEAR --> PROD3["✅ Production"]

    subgraph "Safety"
        ALARM["🚨 Auto-rollback on alarm"]
        BAKE["⏰ Bake time between stages"]
    end

    style ONEBOX fill:#ffd43b,color:#333
    style PROD3 fill:#51cf66,color:#fff
```

---

## Mapping → NestJS

| Amazon | NestJS Implementation |
|---|---|
| **SOA / Microservices** | `@nestjs/microservices` |
| **DynamoDB** | `@aws-sdk/client-dynamodb` / TypeORM |
| **SQS/SNS** | `@nestjs/microservices` + `@aws-sdk/client-sqs` |
| **ElastiCache** | `@nestjs/cache-manager` + ioredis |
| **EventBridge** | Custom event bus / `@nestjs/event-emitter` |
| **Lambda** | NestJS on Lambda (`@nestjs/platform-express`) |
| **CodePipeline** | GitHub Actions + ArgoCD |
| **Two-pizza teams** | NestJS module per domain |
