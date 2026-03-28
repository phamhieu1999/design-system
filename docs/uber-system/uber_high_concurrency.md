# Uber - Xử Lý Đồng Thời Cao & Geospatial

> Millions location updates/giây, real-time matching, surge pricing trên 10,000+ cities.

---

## 1. Ride Matching — The Core Problem

```mermaid
sequenceDiagram
    actor Rider as 👤 Rider
    participant API as API Gateway
    participant Match as Matching Service
    participant Location as Location Service
    participant Supply as Supply Service
    participant ETA as ETA Service
    participant Dispatch as Dispatch Service
    actor Driver as 🚗 Driver

    Rider->>API: Request ride (pickup, dropoff)
    API->>Match: Find driver

    Match->>Location: Get nearby drivers<br/>(H3 cell + kRing neighbors)
    Location-->>Match: Candidate list (N drivers)

    Match->>ETA: Calculate ETA for each
    ETA-->>Match: ETAs (via road network + traffic)

    Match->>Supply: Check driver eligibility
    Supply-->>Match: Filtered candidates

    Match->>Match: Rank by ETA + score
    Match->>Dispatch: Dispatch to best driver

    Dispatch->>Driver: 🔔 Ride offer (15s timeout)

    alt Driver accepts
        Driver->>Dispatch: ✅ Accept
        Dispatch->>Rider: 🚗 Driver assigned!
    else Driver declines / timeout
        Driver->>Dispatch: ❌ Decline
        Dispatch->>Match: Try next driver
    end
```

---

## 2. H3 — Hexagonal Spatial Index

```mermaid
graph TB
    subgraph "H3 Hexagonal Grid"
        EARTH["🌍 Earth Surface"]
        EARTH --> RES0["Resolution 0<br/>(122 base cells)"]
        RES0 --> RES4["Resolution 4<br/>(~1,770 km²)"]
        RES4 --> RES7["Resolution 7<br/>(~5.2 km²)<br/>City-level pricing"]
        RES7 --> RES9["Resolution 9<br/>(~0.1 km²)<br/>Block-level matching"]
        RES9 --> RES15["Resolution 15<br/>(~0.9 m²)<br/>Precise location"]
    end

    subgraph "Why Hexagons?"
        HEX1["✅ All 6 neighbors equidistant<br/>(Squares: corner vs edge ≠)"]
        HEX2["✅ No edge aliasing"]
        HEX3["✅ Better approximation of circles"]
        HEX4["✅ 64-bit integer ID (fast)"]
    end

    style RES9 fill:#e64980,color:#fff
```

### Driver Location Index

```mermaid
flowchart TB
    GPS["📡 Driver GPS<br/>(Every 4 seconds)"] --> H3_CONVERT["Convert lat/lng<br/>→ H3 cell ID"]
    H3_CONVERT --> REDIS_SET["Redis GeoSet<br/>Key: h3_cell_id<br/>Value: driver_id + metadata"]

    RIDE_REQ["🙋 Ride Request"] --> H3_RIDER["Convert rider location<br/>→ H3 cell ID"]
    H3_RIDER --> KRING["kRing(cell, k=2)<br/>→ Get cell + neighbors"]
    KRING --> REDIS_GET["Query Redis for<br/>drivers in cells"]
    REDIS_GET --> CANDIDATES["📋 Candidate drivers"]

    style KRING fill:#4c6ef5,color:#fff
```

---

## 3. Surge Pricing — Dynamic Marketplace

```mermaid
flowchart TB
    subgraph "Real-time Data"
        DEMAND["📈 Ride Requests<br/>(per H3 cell)"]
        SUPPLY2["🚗 Available Drivers<br/>(per H3 cell)"]
    end

    DEMAND & SUPPLY2 --> KAFKA17["📨 Kafka"]
    KAFKA17 --> FLINK5["⚡ Flink<br/>(Sliding window: 5min)"]

    FLINK5 --> RATIO["📊 Supply/Demand Ratio"]
    RATIO --> SURGE_CALC{"Ratio analysis"}

    SURGE_CALC -->|"Supply >> Demand"| NO_SURGE["1.0x (No surge)"]
    SURGE_CALC -->|"Demand > Supply"| SURGE["1.2x - 3.0x+ Surge"]
    SURGE_CALC -->|"Critical shortage"| HIGH_SURGE["🔴 High surge<br/>(Attract distant drivers)"]

    SURGE --> CACHE2["Redis Cache<br/>(Per H3 cell)"]
    CACHE2 --> PRICE_API["💰 Price shown to rider"]

    subgraph "Surge Goals"
        G1["1️⃣ Attract more drivers to area"]
        G2["2️⃣ Reduce demand (some riders wait)"]
        G3["3️⃣ Clear marketplace faster"]
    end

    style SURGE fill:#ffd43b,color:#333
    style HIGH_SURGE fill:#ff6b6b,color:#fff
```

---

## 4. ETA Prediction

```mermaid
flowchart TB
    ORIGIN3["📍 Origin"] --> ROUTING["🗺️ Routing Engine"]

    ROUTING --> FACTORS["Factors:<br/>• Road network graph<br/>• Real-time traffic<br/>• Historical patterns<br/>• Time of day<br/>• Weather<br/>• Events"]

    FACTORS --> ML5["🤖 ML Model<br/>(XGBoost / Deep Learning)"]
    ML5 --> ETA2["⏱️ ETA: 7 minutes"]

    subgraph "Scale"
        S_ETA["Millions of ETA calculations/second"]
        S_ACC["Accuracy: < 1 min error for short trips"]
    end

    style ML5 fill:#e64980,color:#fff
```

---

## 5. Handling Peak Events (New Year's, Concert Endings)

```mermaid
flowchart TB
    EVENT2["🎉 Peak Event<br/>(100x normal demand)"] --> STRATEGIES2

    subgraph "Pre-event"
        PRE1["📊 Predict demand from historical"]
        PRE2["📢 Incentivize drivers to be online"]
        PRE3["⬆️ Pre-scale infrastructure"]
    end

    subgraph "During event"
        DUR1["💰 Surge pricing (attract drivers)"]
        DUR2["🔄 Ride pooling priority"]
        DUR3["⚖️ Load shedding (drop non-critical)"]
        DUR4["📱 Queuing system for riders"]
    end

    subgraph "Post-event"
        POST1["📈 Analytics"]
        POST2["💡 Improve prediction model"]
    end

    STRATEGIES2 --> PRE1 & PRE2 & PRE3 & DUR1 & DUR2 & DUR3 & DUR4
```

---

## 6. So Sánh Concurrency: Uber vs Others

| Aspect | Uber | YouTube | Netflix | WhatsApp |
|---|---|---|---|---|
| **Challenge** | Geospatial real-time matching | Video processing | Bandwidth | Connection count |
| **Data model** | Location streams | Video uploads | Content catalog | Messages |
| **Peak metric** | Millions GPS/sec | 500+ hrs uploaded/min | 500M+ hrs/day | 100B+ msgs/day |
| **Key tech** | H3 + Redis + Flink | Vitess + CDN | Open Connect | BEAM actors |
| **Latency** | < 3s (match) | < 1s (start play) | < 1s (start play) | < 100ms (delivery) |

---

## Mapping → NestJS

| Pattern | Uber | NestJS Implementation |
|---|---|---|
| **H3 geospatial** | H3 hex grid | `h3-js` npm package |
| **Driver location** | Redis GeoSet | Redis `GEOADD` + `GEORADIUS` |
| **Ride matching** | Scoring + dispatch | Custom service + BullMQ |
| **Surge pricing** | Flink streaming | Kafka consumer + sliding window in Redis |
| **ETA calculation** | ML + graph routing | OSRM (Open Source Routing) + `@nestjs/microservices` |
| **Real-time tracking** | WebSocket push | `@nestjs/websockets` + Socket.IO |
