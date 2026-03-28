# Netflix - Deployment & Architecture

> Netflix phục vụ **260M+ subscribers** tại 190+ quốc gia — pioneer của microservices & cloud-native.

---

## 1. Quy Mô

| Metric | Giá trị |
|---|---|
| Paid subscribers | 260M+ |
| Countries | 190+ |
| Streaming hours/day | 500M+ |
| Bandwidth | ~15% internet traffic toàn cầu |
| Microservices | 1,000+ |
| API calls/day | Hàng tỷ |
| Titles in catalog | 15,000+ |

---

## 2. Technology Stack

```mermaid
graph TB
    subgraph "Client Layer"
        TV["📺 Smart TV"]
        MOBILE3["📱 Mobile (iOS/Android)"]
        WEB4["🌐 Web Browser"]
        CONSOLE["🎮 Game Console"]
    end

    subgraph "Edge (AWS)"
        ZUUL["🚪 Zuul 2<br/>(API Gateway)"]
    end

    subgraph "Backend Services (AWS)"
        JAVA["☕ Java / Spring Boot<br/>(Core services, Virtual Threads)"]
        GRAPHQL2["📊 GraphQL Federation<br/>(API orchestration)"]
        NODE["Node.js<br/>(UI rendering)"]
        PYTHON2["🐍 Python<br/>(ML/Data pipelines)"]
    end

    subgraph "Service Mesh"
        EUREKA["📖 Eureka<br/>(Service Discovery)"]
        ENVOY["🔀 Envoy Proxy<br/>(Service Mesh)"]
        R4J["🔌 Resilience4j<br/>(Circuit Breaker)"]
    end

    subgraph "Data Layer"
        CASS["📀 Cassandra<br/>(User data, billions rows)"]
        AURORA["🐦 Amazon Aurora<br/>(Transactional)"]
        ES3["🔍 Elasticsearch<br/>(Search)"]
        REDIS4["🔴 Redis<br/>(Cache)"]
        S3_2["📦 S3<br/>(Content storage)"]
    end

    subgraph "Streaming"
        KAFKA13["📨 Apache Kafka<br/>(Trillions events/day)"]
        FLINK2["⚡ Apache Flink<br/>(Real-time processing)"]
    end

    subgraph "Infrastructure"
        TITUS["📦 Titus<br/>(Container platform)"]
        SPINNAKER["🚀 Spinnaker<br/>(Multi-region CD)"]
        AWS["☁️ AWS<br/>(All compute)"]
    end

    TV & MOBILE3 & WEB4 & CONSOLE --> ZUUL
    ZUUL --> JAVA & GRAPHQL2
    JAVA --> EUREKA & R4J
    JAVA --> CASS & AURORA & REDIS4
    JAVA --> KAFKA13
```

### Tech Stack Evolution

```mermaid
timeline
    title Netflix Tech Stack Evolution
    2007 : Monolithic Java (data center)
         : Oracle DB, single deploy
    2009 : "The Great Migration" to AWS
         : Cassandra replaces Oracle
    2012 : Netflix OSS born
         : Eureka, Hystrix, Zuul, Ribbon
    2015 : 1000+ microservices
         : Spinnaker for CI/CD
    2018 : Titus container platform
         : RxJava everywhere
    2021 : GraphQL Federation
         : Hystrix → Resilience4j
    2024 : Spring Boot 3+ / JDK 21+
         : Virtual Threads replace RxJava
         : Envoy service mesh
```

---

## 3. System Architecture — Dual Plane

```mermaid
graph TB
    subgraph "Control Plane (AWS)"
        USER2["📱 User"] --> ZUUL2["Zuul API Gateway"]
        ZUUL2 --> AUTH["🔑 Auth Service"]
        ZUUL2 --> PROFILE["👤 Profile Service"]
        ZUUL2 --> RECOMMEND2["🤖 Recommendation"]
        ZUUL2 --> SEARCH2["🔍 Search"]
        ZUUL2 --> BILLING["💳 Billing"]
        ZUUL2 --> PLAYBACK["▶️ Playback Service"]

        PLAYBACK --> STEERING["🧭 Steering Service<br/>(Pick best OCA)"]
    end

    subgraph "Data Plane (Open Connect CDN)"
        STEERING --> OCA1["📦 OCA @ ISP-A<br/>(Inside ISP datacenter)"]
        STEERING --> OCA2["📦 OCA @ ISP-B"]
        STEERING --> OCA3["📦 OCA @ IXP"]
        
        OCA1 --> STREAM["🎬 Video Stream<br/>(Direct to user)"]
    end

    USER2 -.->|"Video data<br/>(bypass internet)"| OCA1

    style OCA1 fill:#e64980,color:#fff
    style STREAM fill:#51cf66,color:#fff
```

**Key insight:** Control plane (auth, recommendations, billing) ở AWS. Data plane (actual video bytes) ở Open Connect CDN **bên trong ISP** → bypasses public internet.

---

## 4. Deployment — Spinnaker + Titus

```mermaid
flowchart TB
    COMMIT2["👨‍💻 Commit"] --> BUILD3["🔨 Build + Unit Tests"]
    BUILD3 --> BAKE["🍳 Bake AMI / Container Image"]
    BAKE --> SPINNAKER2["🚀 Spinnaker"]
    
    SPINNAKER2 --> CANARY3["🐤 Canary Deploy<br/>(Small % traffic)"]
    CANARY3 --> ANALYZE2["📊 Automated Canary Analysis<br/>(ACA - compare metrics)"]
    
    ANALYZE2 -->|"Canary OK"| ROLL["📈 Rolling Deploy<br/>(Region by region)"]
    ANALYZE2 -->|"Canary BAD"| ROLLBACK2["⏮️ Auto Rollback"]
    
    ROLL --> PROD2["✅ Production (All regions)"]

    subgraph "Titus (Container Platform)"
        TITUS2["Netflix Titus<br/>• Built on Docker + AWS<br/>• Auto-scaling<br/>• Resource isolation<br/>• Multi-tenant"]
    end

    SPINNAKER2 --> TITUS2

    style CANARY3 fill:#ffd43b,color:#333
    style PROD2 fill:#51cf66,color:#fff
    style ROLLBACK2 fill:#ff6b6b,color:#fff
```

| Tool | Purpose |
|---|---|
| **Spinnaker** | Multi-region CD, canary analysis, automated rollback |
| **Titus** | Container orchestration (Netflix's internal K8s alternative) |
| **ACA** | Automated Canary Analysis — statistical comparison of metrics |
| **Bake** | Immutable server images (AMI / container) |

---

## 5. Netflix OSS — Open Source Contributions

| Tool | Purpose | Status |
|---|---|---|
| **Eureka** | Service Discovery | Active — still core |
| **Zuul 2** | API Gateway (async) | Active |
| **Hystrix** | Circuit Breaker | ❌ Retired → Resilience4j |
| **Ribbon** | Client-side Load Balancer | ❌ Retired → Spring Cloud LB |
| **Spinnaker** | Continuous Delivery | Active — CNCF project |
| **Chaos Monkey** | Random instance termination | Active |
| **Titus** | Container platform | Active |
| **Conductor** | Workflow orchestration | Active |
| **Zuul** | Edge routing | Active |

---

## Mapping → NestJS

| Netflix | NestJS Implementation |
|---|---|
| **Java + Spring Boot** | NestJS (TypeScript) |
| **Zuul (API Gateway)** | `@nestjs/microservices` + Kong/Nginx |
| **Eureka (Discovery)** | Consul / `nestjs-consul` |
| **Resilience4j** | `opossum` (circuit breaker) |
| **Spinnaker (CD)** | GitHub Actions + ArgoCD |
| **Titus (Containers)** | Kubernetes + Helm |
| **GraphQL** | `@nestjs/graphql` + Apollo Federation |
| **Kafka** | `@nestjs/microservices` Kafka transport |
| **Cassandra** | `cassandra-driver` / TypeORM |
