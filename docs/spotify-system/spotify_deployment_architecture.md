# Spotify - Deployment & Architecture

> Spotify phục vụ **600M+ users**, **100M+ tracks**, **5M+ podcasts** tại 180+ markets.

---

## 1. Quy Mô

| Metric | Giá trị |
|---|---|
| Monthly Active Users | 600M+ |
| Premium subscribers | 230M+ |
| Tracks | 100M+ |
| Podcasts | 5M+ |
| Markets | 180+ |
| Microservices | 1,000+ |
| Playlists created | 4B+ |

---

## 2. Technology Stack

```mermaid
graph TB
    subgraph "Client"
        WEB8["🌐 Web (React)"]
        DESK["🖥️ Desktop (Electron/CEF)"]
        IOS5["📱 iOS (Swift)"]
        AND4["📱 Android (Kotlin)"]
    end

    subgraph "Edge"
        CDN7["🌍 Google Cloud CDN"]
        LB7["⚖️ Cloud Load Balancer"]
    end

    subgraph "Backend"
        JAVA6["☕ Java + Spring<br/>(Core services)"]
        SCALA3["📐 Scala<br/>(Data pipelines)"]
        PY3["🐍 Python<br/>(ML models)"]
        NODE2["🟢 Node.js<br/>(Lightweight APIs)"]
    end

    subgraph "Data"
        CASS3["📀 Cassandra<br/>(User activity)"]
        PG2["🐘 PostgreSQL<br/>(Metadata)"]
        GCS2["📦 Cloud Storage<br/>(Audio files)"]
        BQ["📊 BigQuery<br/>(Analytics)"]
    end

    subgraph "Streaming & ML"
        KAFKA21["📨 Apache Kafka<br/>(Events backbone)"]
        BEAM["⚡ Apache Beam / Scio<br/>(Data processing)"]
        TF3["🧠 TensorFlow<br/>(Recommendation)"]
    end

    subgraph "Infrastructure"
        GKE["☸️ GKE (Kubernetes)"]
        BACKSTAGE["🎭 Backstage<br/>(Developer portal)"]
    end

    WEB8 & DESK & IOS5 & AND4 --> CDN7 --> LB7
    LB7 --> JAVA6
    JAVA6 --> CASS3 & PG2 & KAFKA21
```

---

## 3. Architecture Evolution

```mermaid
timeline
    title Spotify Architecture Evolution
    2006-2008 : Desktop-first (C++/Python)
              : Peer-to-peer streaming
    2010-2012 : Backend services (Java)
              : PostgreSQL + Cassandra
    2013-2015 : Microservices explosion
              : 800+ services
    2016-2018 : Migration to GCP
              : Kubernetes adoption
    2019-2020 : Backstage created
              : "Golden paths" for devs
    2021-2024 : Podcasts & Audiobooks
              : AI DJ, AI playlists
              : Backstage → CNCF project
```

---

## 4. Squad Model — Organization Design

```mermaid
graph TB
    subgraph "Tribe: Music Player"
        SQ1["🎵 Squad: Playback"]
        SQ2["🔍 Squad: Search"]
        SQ3["📻 Squad: Radio"]
    end

    subgraph "Tribe: Recommendations"
        SQ4["🤖 Squad: Discover Weekly"]
        SQ5["📊 Squad: Home Page"]
        SQ6["🎧 Squad: Made For You"]
    end

    subgraph "Cross-cutting"
        CH1["📚 Chapter: Backend Engineers"]
        CH2["📚 Chapter: Data Scientists"]
        GUILD["🏰 Guild: ML Guild<br/>(Voluntary, company-wide)"]
    end

    SQ1 -.-> CH1
    SQ4 -.-> CH1
    SQ4 -.-> CH2
    SQ5 -.-> CH2

    style SQ4 fill:#1DB954,color:#fff
```

| Concept | Description |
|---|---|
| **Squad** | 6-12 people, autonomous, owns feature end-to-end |
| **Tribe** | Group of related squads (40-150 people) |
| **Chapter** | Same-role people across squads (mentorship) |
| **Guild** | Voluntary community of interest (knowledge sharing) |

---

## 5. Backstage — Developer Portal

```mermaid
flowchart TB
    DEV2["👨‍💻 Developer"] --> BACKSTAGE2["🎭 Backstage Portal"]

    BACKSTAGE2 --> CATALOG["📦 Service Catalog<br/>(All 1000+ services)"]
    BACKSTAGE2 --> TEMPLATES["📋 Templates<br/>(Create new service in minutes)"]
    BACKSTAGE2 --> DOCS["📖 TechDocs<br/>(Living documentation)"]
    BACKSTAGE2 --> PLUGINS["🔌 Plugins<br/>(CI/CD, K8s, PagerDuty)"]

    NOTE15["Backstage is now a<br/>CNCF Incubating project<br/>Used by: Expedia, Netflix, IKEA"]

    style BACKSTAGE2 fill:#1DB954,color:#fff
```

---

## 6. Deployment

```mermaid
flowchart TB
    CODE6["👨‍💻 Code"] --> CI4["🧪 CI (Tests)"]
    CI4 --> BUILD6["🔨 Build container image"]
    BUILD6 --> DEPLOY4["🚀 Deploy to GKE<br/>(Blue-green / canary)"]
    DEPLOY4 --> MONITOR3["📊 Monitor metrics"]
    MONITOR3 -->|"Issues"| ROLLBACK4["⏮️ Automated rollback"]
```

---

## Mapping → NestJS

| Spotify | NestJS Implementation |
|---|---|
| **Java + Spring** | NestJS (TypeScript) |
| **Scala/Beam** | BullMQ workers / Kafka consumers |
| **Cassandra** | `cassandra-driver` / ScyllaDB |
| **Backstage** | Backstage (directly reusable!) |
| **GKE** | Any Kubernetes cluster |
| **Squad model** | NestJS module per squad/domain |
| **Kafka** | `@nestjs/microservices` Kafka transport |
