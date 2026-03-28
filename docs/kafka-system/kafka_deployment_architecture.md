# Apache Kafka - Deployment & Architecture

> LinkedIn tạo ra Kafka xử lý **trillions messages/ngày**, nền tảng event streaming #1 thế giới.

---

## 1. Quy Mô (LinkedIn Reference)

| Metric | Giá trị |
|---|---|
| Messages/day (LinkedIn) | 7 trillion+ |
| Throughput | Millions msgs/sec per cluster |
| Companies using | LinkedIn, Uber, Spotify, Netflix, Stripe, Airbnb |
| Open source since | 2011 (Apache) |
| Creator | Jay Kreps, Neha Narkhede (LinkedIn) |

---

## 2. Core Architecture

```mermaid
graph TB
    subgraph "Producers"
        P1["🏢 Service A"]
        P2["📱 App Events"]
        P3["📊 Metrics"]
    end

    subgraph "Kafka Cluster"
        B1["Broker 1"]
        B2["Broker 2"]
        B3["Broker 3"]
        CTRL["🧠 Controller<br/>(KRaft / Raft consensus)"]
    end

    subgraph "Consumers"
        CG1["Consumer Group A<br/>(Order Service)"]
        CG2["Consumer Group B<br/>(Analytics)"]
        CG3["Consumer Group C<br/>(Search Index)"]
    end

    P1 & P2 & P3 --> B1 & B2 & B3
    B1 & B2 & B3 --> CG1 & CG2 & CG3

    style CTRL fill:#e64980,color:#fff
```

---

## 3. The Distributed Log — Core Concept

```mermaid
flowchart TB
    subgraph "Topic: 'orders' (3 partitions)"
        P0["Partition 0<br/>|0|1|2|3|4|5|6|7| →<br/>(Broker 1 = Leader)"]
        P1_2["Partition 1<br/>|0|1|2|3|4|5| →<br/>(Broker 2 = Leader)"]
        P2_2["Partition 2<br/>|0|1|2|3|4|5|6|7|8| →<br/>(Broker 3 = Leader)"]
    end

    subgraph "Key Properties"
        K1["📝 Append-only (immutable)"]
        K2["🔢 Ordered within partition"]
        K3["⏰ Retention-based (time or size)"]
        K4["🔄 Replayable (consumers track offset)"]
    end

    style P0 fill:#4c6ef5,color:#fff
```

### Why Append-Only Log is Fast

| Operation | Disk | Speed |
|---|---|---|
| Random write | Seek to position | ~10ms |
| Sequential write (Kafka) | Append to end | ~0.03ms |
| **Result** | Kafka = 300x faster | Sequential I/O wins |

**Zero-copy transfer:** Kafka uses `sendfile()` syscall → data goes from disk → network socket, bypassing user space → maximizes throughput.

---

## 4. Replication & ISR

```mermaid
flowchart TB
    subgraph "Partition 0 (Replication Factor = 3)"
        LEADER2["🟢 Leader (Broker 1)<br/>Offset: 0,1,2,3,4,5"]
        FOLLOWER1["🔵 Follower (Broker 2)<br/>Offset: 0,1,2,3,4,5<br/>✅ In ISR"]
        FOLLOWER2["🔵 Follower (Broker 3)<br/>Offset: 0,1,2,3<br/>⚠️ Lagging, removed from ISR"]
    end

    PRODUCER2["Producer<br/>(acks=all)"] --> LEADER2
    LEADER2 --> FOLLOWER1
    LEADER2 --> FOLLOWER2

    subgraph "acks configuration"
        ACKS0["acks=0: Fire-and-forget (fastest, risky)"]
        ACKS1["acks=1: Leader confirms (balanced)"]
        ACKSALL["acks=all: All ISR confirm (safest)"]
    end

    style LEADER2 fill:#51cf66,color:#fff
    style FOLLOWER2 fill:#ffd43b,color:#333
```

---

## 5. ZooKeeper → KRaft Evolution

```mermaid
timeline
    title Kafka Metadata Management
    2011-2022 : ZooKeeper mode
              : External ZK cluster manages metadata
              : Controller elected via ZK
    2022 : KRaft (Early access)
         : Raft consensus built into Kafka
    2023-2024 : KRaft GA (default)
              : ZooKeeper deprecated
    2025+ : ZooKeeper removed
          : KRaft is the only mode
```

```mermaid
flowchart LR
    subgraph "Old: ZooKeeper"
        ZK_KAFKA["Kafka Cluster"] --> ZK["ZooKeeper Ensemble<br/>(Separate cluster!)"]
    end

    subgraph "New: KRaft"
        KRAFT_KAFKA["Kafka Cluster<br/>(Self-contained!)"]
        KRAFT_CTRL["🧠 Controller Quorum<br/>(Raft consensus, internal)"]
        KRAFT_KAFKA --> KRAFT_CTRL
    end

    style KRAFT_KAFKA fill:#51cf66,color:#fff
```

---

## 6. Deployment Topology

```mermaid
graph TB
    subgraph "Production Cluster"
        B1_2["Broker 1<br/>(Controller)"]
        B2_2["Broker 2<br/>(Controller)"]
        B3_2["Broker 3<br/>(Controller)"]
        B4["Broker 4"]
        B5["Broker 5"]
        B6["Broker 6"]
    end

    subgraph "Support Services"
        SR["📋 Schema Registry"]
        KC["🔌 Kafka Connect"]
        KSQL["📊 ksqlDB"]
        UI2["📱 Kafka UI"]
    end

    subgraph "Monitoring"
        JMX["📊 JMX Metrics"]
        PROM2["📈 Prometheus + Grafana"]
        CRUISE["🚢 Cruise Control<br/>(Auto-rebalancing)"]
    end
```

---

## Mapping → NestJS

| Kafka | NestJS Implementation |
|---|---|
| **Producer** | `@nestjs/microservices` Kafka transport |
| **Consumer** | `@MessagePattern()` / `@EventPattern()` |
| **Schema Registry** | `@kafkajs/confluent-schema-registry` |
| **KRaft cluster** | Docker Compose / K8s Helm chart |
| **Connect** | Use connectors for DB sync |
| **Monitoring** | `kafkajs` + Prometheus exporter |
