# 🔴 Redis System - Phân Tích Kiến Trúc

> Tài liệu phân tích Redis — Bộ nhớ đệm (In-memory Data Store) phổ biến nhất thế giới.

## Danh Sách Tài Liệu

| # | File | Nội dung |
|---|---|---|
| 1 | [redis_deployment_architecture.md](./redis_deployment_architecture.md) | Single-threaded, epoll, jemalloc, Internal Data Structures (SDS, listpack) |
| 2 | [redis_high_concurrency.md](./redis_high_concurrency.md) | Persistence (RDB/AOF), Master-Replica, Sentinel HA, Cluster (16384 Slots) |
| 3 | [redis_security.md](./redis_security.md) | Access Control Lists (ACLs), Eviction Policies (LRU vs LFU), OOM Protection |
| 4 | [redis_subsystems.md](./redis_subsystems.md) | Pub/Sub, Redis Streams, Distributed Lock, HyperLogLog, Geospatial |

## Tổng Quan

```
🔴 Redis
├── 🏗️ Infrastructure
│   ├── Single-Threaded Event Loop (epoll) ──── redis_deployment_architecture.md
│   ├── jemalloc Memory Allocation ───────────── redis_deployment_architecture.md
│   └── SDS, quicklist, listpack, skiplist ───── redis_deployment_architecture.md
│
├── ⚡ Performance & Scalability
│   ├── Redis Master-Replica ─────────────────── redis_high_concurrency.md
│   ├── Sentinel (High Availability) ─────────── redis_high_concurrency.md
│   └── Redis Cluster (Sharding đằng sau hash) ─ redis_high_concurrency.md
│
├── 🔒 Security & Memory
│   ├── RDB / AOF Persistence ────────────────── redis_high_concurrency.md
│   ├── LRU / LFU Eviction Policies ──────────── redis_security.md
│   └── O(N) Dangerous Commands (FLUSHALL) ───── redis_security.md
│
└── 📦 Subsystems
    ├── Pub/Sub (Ephemeral) vs Streams (Log) ── redis_subsystems.md
    ├── Distributed Lock (LUA Script) ────────── redis_subsystems.md
    └── Geo-Spatial & HyperLogLog ────────────── redis_subsystems.md
```

## Key Patterns (Áp dụng cho NestJS)
| Pattern | NestJS Implementation / Use Case |
|---|---|
| **Caching Layer** | @nestjs/cache-manager làm In-memory lookup nhanh (5ms). |
| **Distributed Lock** | Tránh Race condition trong microservices (VD: Trừ tiền 2 nơi dùng Redlock). |
| **Job Queue** | Sử dụng kết hợp **BullMQ** (list, hash, zset nội bộ của redis). |
| **Streams** | Làm Message Broker gọn nhẹ (thay thế Kafka nếu Scale tầm trung). |
