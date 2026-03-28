# Uber - Deployment & Architecture

> Uber phục vụ **130M+ monthly users**, **6.9B+ trips/năm** tại 10,000+ cities, 70+ countries.

---

## 1. Quy Mô

| Metric | Giá trị |
|---|---|
| Monthly Active Users | 130M+ |
| Trips per year | 6.9B+ |
| Cities | 10,000+ |
| Countries | 70+ |
| Microservices | 4,000+ |
| Location updates/sec | Millions |
| Revenue (2024) | $40B+ |

---

## 2. Technology Stack

```mermaid
graph TB
    subgraph "Client"
        RIDER["📱 Rider App<br/>(React Native)"]
        DRIVER["🚗 Driver App<br/>(Native iOS/Android)"]
        EATS["🍔 Uber Eats"]
        WEB6["🌐 Web"]
    end

    subgraph "Edge"
        API_GW["🚪 API Gateway<br/>(Go-based)"]
    end

    subgraph "Backend (Go + Java)"
        GO3["🦫 Go<br/>(High-perf: matching,<br/>location, dispatch)"]
        JAVA3["☕ Java<br/>(Business logic,<br/>payments, complex flows)"]
    end

    subgraph "Data Infrastructure"
        MYSQL5["🐬 MySQL<br/>(Schemaless layer)"]
        CASS2["📀 Cassandra<br/>(High-write workloads)"]
        REDIS5["🔴 Redis<br/>(Real-time cache)"]
        SPANNER3["🌐 Google Spanner<br/>(Distributed SQL)"]
    end

    subgraph "Streaming & Analytics"
        KAFKA16["📨 Apache Kafka<br/>(Central messaging hub)"]
        FLINK4["⚡ Apache Flink<br/>(Real-time processing)"]
        PINOT["📊 Apache Pinot<br/>(Real-time analytics)"]
        SPARK4["🔥 Apache Spark<br/>(Batch)"]
        PRESTO3["🔍 Presto<br/>(Interactive SQL)"]
    end

    subgraph "ML"
        MICHELANGELO["🎨 Michelangelo<br/>(ML Platform)"]
        TF2["🧠 TensorFlow/PyTorch"]
    end

    RIDER & DRIVER --> API_GW --> GO3 & JAVA3
    GO3 --> REDIS5 & KAFKA16
    JAVA3 --> MYSQL5 & CASS2
```

---

## 3. Architecture Evolution

```mermaid
timeline
    title Uber Architecture Evolution
    2010-2012 : Monolith (Python)
              : Single MySQL database
    2013-2014 : SOA Migration
              : Python → Go/Java
              : Schemaless (custom MySQL wrapper)
    2015-2017 : Microservices explosion
              : 2000+ services
              : Dependency chaos
    2018-2020 : DOMA introduced
              : Domain-oriented organization
              : Gateway pattern
    2021-2024 : Cloud migration (GCP)
              : Service mesh (Envoy)
              : Monorepo consolidation
```

---

## 4. DOMA — Domain-Oriented Microservice Architecture

```mermaid
graph TB
    subgraph "Layer 5: Edge (Gateway)"
        EDGE2["API Gateway (Go)"]
    end

    subgraph "Layer 4: Product"
        RIDER2["Rider Domain"] & DRIVER2["Driver Domain"] & EATS2["Eats Domain"]
    end

    subgraph "Layer 3: Business"
        TRIP["Trip Domain"] & PRICING["Pricing Domain"] & MATCH["Matching Domain"]
    end

    subgraph "Layer 2: Platform"
        PAYMENT["Payment Domain"] & NOTIFICATION2["Notification Domain"] & IDENTITY["Identity Domain"]
    end

    subgraph "Layer 1: Infrastructure"
        STORAGE2["Storage Platform"] & MESSAGING2["Messaging Platform"] & OBSERV["Observability"]
    end

    EDGE2 --> RIDER2 & DRIVER2 & EATS2
    RIDER2 & DRIVER2 & EATS2 --> TRIP & PRICING & MATCH
    TRIP & PRICING & MATCH --> PAYMENT & NOTIFICATION2 & IDENTITY
    PAYMENT & NOTIFICATION2 & IDENTITY --> STORAGE2 & MESSAGING2 & OBSERV

    style EDGE2 fill:#4c6ef5,color:#fff
```

### DOMA Principles

| Principle | Description |
|---|---|
| **Domains** | Group related services (e.g., Payment Domain has 30+ microservices) |
| **Layers** | Upper can depend on lower, NEVER reverse |
| **Gateways** | Single entry point per domain — hide internal complexity |
| **Extensions** | Customize domains without modifying core |

---

## 5. Deployment

```mermaid
flowchart TB
    CODE3["👨‍💻 Code (Monorepo)"] --> CI2["🔨 CI: Build + Test"]
    CI2 --> UPDC["📦 Up CD<br/>(Uber's internal CD)"]
    UPDC --> CANARY5["🐤 Canary Deploy<br/>(Shadow traffic)"]
    CANARY5 --> METRIC["📊 Automated metric check"]
    METRIC -->|"OK"| ROLLING2["📈 Rolling deploy<br/>(Region by region)"]
    METRIC -->|"Degraded"| ROLLBACK3["⏮️ Auto rollback"]

    subgraph "Container Platform"
        PELOTON["📦 Peloton<br/>(Uber's resource mgr)<br/>→ migrating to K8s"]
    end

    style CANARY5 fill:#ffd43b,color:#333
```

---

## Mapping → NestJS

| Uber | NestJS Implementation |
|---|---|
| **DOMA (Domains)** | NestJS Modules with gateway pattern |
| **Go API Gateway** | `@nestjs/microservices` + Kong |
| **Kafka** | `@nestjs/microservices` Kafka transport |
| **Schemaless** | TypeORM + PostgreSQL |
| **Redis cache** | `@nestjs/cache-manager` + ioredis |
| **Flink** | BullMQ workers (simplified) |
| **Michelangelo (ML)** | Python service via gRPC |
| **Peloton/K8s** | Kubernetes + Helm |
