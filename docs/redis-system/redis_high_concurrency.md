# Redis - Xử Lý Đồng Thời Cao & Scaling

> Scaling read/write, Persistent data (RDB/AOF), Redis Cluster.

---

## 1. Persistence: RDB vs AOF

Làm sao in-memory DB lại không mất dữ liệu khi sập nguồn?

```mermaid
flowchart TB
    subgraph "Memory"
        RAM["Redis RAM Space"]
    end

    RAM -->|"Fork() background"| RDB["📀 RDB (Snapshot)<br/>File dump.rdb gọn nhẹ"]
    RAM -->|"Log every write"| AOF["📝 AOF (Append Only File)<br/>File text chứa tuần tự script"]

    subgraph "AOF fsync policies"
        F1["always: Mọi lệnh → Rất chậm, an toàn"]
        F2["everysec: 1 giây/lần → Default, mất quá 1s data"]
        F3["no: OS tự quyết định → Nhanh, rủi ro cao"]
    end

    RDB --> MIXED["Hybrid Mode (Tiêu chuẩn mới):<br/>RDB preamble + AOF log"]
    AOF --> MIXED
```

---

## 2. Replication (Master-Replica)

```mermaid
sequenceDiagram
    participant Client
    participant Master
    participant Replica

    Client->>Master: SET key "value"
    Master->>Master: Execute in RAM
    Master-->>Client: ✅ OK (Async replication!)
    
    Master->>Replica: Send stream of commands
    Replica->>Replica: Execute command
    
    Note over Master,Replica: WAIT command có thể force<br/>synchronous replication (chậm).
```

---

## 3. High Availability (Redis Sentinel) & Split-Brain

```mermaid
flowchart TB
    subgraph "Data Center"
        M["👑 Master"]
        R1["🔵 Replica 1"]
        R2["🔵 Replica 2"]
    end

    subgraph "Sentinels (Quorum = 2)"
        S1["Sentinel 1"]
        S2["Sentinel 2"]
        S3["Sentinel 3"]
    end

    S1 & S2 & S3 -.->|"PING/Monitoring"| M

    M -.->|"Crash!"| FAIL["Master sập!"]
    FAIL -.-> S1 & S2 & S3
    
    S1 & S2 & S3 -->|"Election"| PROMOTE["Promote R1 lên làm Master mới"]
    PROMOTE --> RECONFIG["Báo R2 follow Master mới"]
```

**Split-Brain Problem:** 
Chỉ nên chạy số lẻ Sentinel (3, 5). Nếu chạy 2 Sentinel, khi đứt cáp mạng ở giữa, 2 bên đều có 1 Sentinel + 1 Redis node → Không bên nào đạt Quorum (2) → Failover không diễn ra, hoặc hệ thống nghĩ có 2 Master (Split-brain). 

---

## 4. Redis Cluster (Horizontal Scaling)

Khi dữ liệu vượt quá RAM của 1 server (VD: 100GB+), phải sharding.

```mermaid
flowchart TB
    APP["Ứng dụng"] --> HASH["CRC16(key) mod 16384"]

    subgraph "16,384 Hash Slots"
        HASH --> S1["Slot 1423"]
        HASH --> S2["Slot 7000"]
        HASH --> S3["Slot ..."]
    end

    subgraph "Redis Cluster Topology"
        M1["Master 1 (Slots 0-5460)"]
        M2["Master 2 (Slots 5461-10922)"]
        M3["Master 3 (Slots 10923-16383)"]
    end

    S1 --> M1
    S2 --> M2
    S3 --> M3

    subgraph "Gossip Protocol"
        M1 -.-> M2 -.-> M3 -.-> M1
    end
```

### Gossip Protocol
Thay vì cần ZooKeeper (như Kafka cũ) để quản lý cluster, các node Redis Cluster tự PING-PONG nói chuyện với nhau (Gossip protocol) qua Cluster Bus port (VD: node port 6379 thì bus port là 16379) để chia sẻ:
- Node nào đang sống/chết.
- Slots nào thuộc node nào.

### Hash Tags
Nếu cần chạy lệnh Multi-key (như `MGET` hoặc Transaction) mà các keys khác Node sẽ báo lỗi.
Giải quyết: Dùng `{tag}` trong key.
Ví dụ: `user:{123}:profile` và `user:{123}:orders` → Redis chỉ hash số `123` → Chắc chắn 2 keys rơi vào cùng 1 Slot và cùng 1 Node.

---

## Mapping → NestJS

| Pattern | NestJS Implementation |
|---|---|
| **Cluster mode** | Bật `ioredis` dùng `new Redis.Cluster([{ host, port }...])` |
| **Hash Tags** | Dịch vụ khi tạo key luôn kẹp ID: `` `${prefix}:{${userId}}` `` |
| **Replication** | Lệnh write về Master, read đẩy sang Replica (ioredis scale reads) |
| **Sentinel** | `ioredis` config `sentinels: [{ host, port }]`, `name: 'mymaster'` |
