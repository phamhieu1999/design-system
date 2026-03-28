# Apache Kafka - Xử Lý Đồng Thời Cao & Exactly-Once

> Trillions msgs/ngày, exactly-once semantics, consumer group scaling.

---

## 1. Exactly-Once Semantics (EOS) — End to End

```mermaid
flowchart TB
    subgraph "Delivery Semantics"
        AT_MOST["At-most-once:<br/>May lose messages<br/>(acks=0, no retry)"]
        AT_LEAST["At-least-once:<br/>May duplicate messages<br/>(acks=all, retry ON)"]
        EXACTLY["Exactly-once:<br/>No loss, no duplicates<br/>(Idempotent + Transactions)"]
    end

    style EXACTLY fill:#51cf66,color:#fff
```

---

## 2. Idempotent Producer

```mermaid
sequenceDiagram
    participant Producer3 as Producer
    participant Broker as Broker (Leader)

    Producer3->>Broker: Send batch (PID=1, Seq=0)
    Broker-->>Producer3: ✅ ACK

    Producer3->>Broker: Send batch (PID=1, Seq=1)
    Note over Producer3,Broker: Network timeout!
    
    Producer3->>Broker: Retry batch (PID=1, Seq=1)
    Broker->>Broker: Seq=1 already seen for PID=1
    Broker-->>Producer3: ✅ ACK (deduplicated!)

    Note over Broker: Only 1 copy in log!
```

```
Config:
  enable.idempotence = true   # Auto-sets acks=all, retries=MAX
  
Broker tracks per partition:
  { ProducerID → last sequence number }
  → Rejects duplicates automatically
```

---

## 3. Transactional Producer (Atomic Multi-Partition Writes)

```mermaid
sequenceDiagram
    participant Producer4 as Transactional Producer
    participant TC as Transaction Coordinator
    participant Part_A as Partition A
    participant Part_B as Partition B

    Producer4->>TC: initTransactions()
    Producer4->>TC: beginTransaction()
    
    Producer4->>Part_A: Send message to topic-A
    Producer4->>Part_B: Send message to topic-B
    Producer4->>TC: sendOffsetsToTransaction()<br/>(commit consumer offsets too!)

    Producer4->>TC: commitTransaction()
    TC->>Part_A: Mark as committed ✅
    TC->>Part_B: Mark as committed ✅

    Note over Part_A,Part_B: Both visible atomically<br/>OR neither (on abort)
```

---

## 4. Consumer Groups & Rebalancing

```mermaid
flowchart TB
    subgraph "Topic: orders (6 partitions)"
        P_0["P0"] 
        P_1["P1"]
        P_2["P2"]
        P_3["P3"]
        P_4["P4"]
        P_5["P5"]
    end

    subgraph "Consumer Group: order-service (3 consumers)"
        C1_2["Consumer 1<br/>→ P0, P1"]
        C2_2["Consumer 2<br/>→ P2, P3"]
        C3_2["Consumer 3<br/>→ P4, P5"]
    end

    P_0 & P_1 --> C1_2
    P_2 & P_3 --> C2_2
    P_4 & P_5 --> C3_2

    NOTE20["Max parallelism = number of partitions<br/>Adding Consumer 4 → one will be idle"]

    style NOTE20 fill:#4c6ef5,color:#fff
```

### Cooperative Rebalancing

```mermaid
flowchart TB
    subgraph "Old: Eager Rebalance"
        E1["⚠️ ALL consumers stop"]
        E2["Reassign ALL partitions"]
        E3["ALL consumers restart"]
        E4["❌ Downtime during rebalance"]
    end

    subgraph "New: Cooperative (Incremental)"
        C1_3["Only affected partitions reassigned"]
        C2_3["Other consumers keep working"]
        C3_3["✅ Minimal disruption"]
    end

    style C3_3 fill:#51cf66,color:#fff
    style E4 fill:#ff6b6b,color:#fff
```

---

## 5. Offset Management

```mermaid
flowchart TB
    CONSUME2["Consumer reads message<br/>offset=42"] --> PROCESS4["Process message"]
    PROCESS4 --> COMMIT["Commit offset=43<br/>(to __consumer_offsets topic)"]

    subgraph "Commit Strategies"
        AUTO["Auto-commit (default):<br/>Periodic, may re-process"]
        MANUAL2["Manual commit:<br/>After processing, precise"]
        TRANSACTIONAL["Transactional commit:<br/>Atomic with produce, EOS"]
    end

    subgraph "On Consumer Restart"
        RESTART["Consumer restarts →<br/>Read last committed offset →<br/>Resume from offset=43"]
    end

    style TRANSACTIONAL fill:#51cf66,color:#fff
```

---

## 6. Performance Tuning

| Parameter | Default | Tuned | Effect |
|---|---|---|---|
| **batch.size** | 16KB | 64KB-256KB | More throughput |
| **linger.ms** | 0 | 5-20 | Batches more messages |
| **compression.type** | none | lz4/snappy | 50-70% less network |
| **buffer.memory** | 32MB | 64-128MB | More buffering |
| **fetch.min.bytes** | 1 | 1KB-1MB | Less consumer overhead |
| **num.partitions** | 1 | 6-12 per topic | More parallelism |

---

## Mapping → NestJS

| Pattern | Kafka | NestJS Implementation |
|---|---|---|
| **Idempotent producer** | `enable.idempotence=true` | KafkaJS config in `ClientKafka` |
| **Transactional** | `transactional.id` | KafkaJS producer transactions |
| **Consumer group** | `group.id` | `@nestjs/microservices` group option |
| **Manual commit** | `autoCommit: false` | Custom `commitOffsets()` in handler |
| **Batch consumption** | `eachBatch` | KafkaJS `eachBatch` mode |
| **Compression** | `compression: lz4` | KafkaJS compression codec |
