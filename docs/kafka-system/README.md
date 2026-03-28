# 📨 Apache Kafka System - Phân Tích Kiến Trúc

> Tài liệu phân tích Apache Kafka — Trillions messages/ngày, Event Streaming Platform.

## Danh Sách Tài Liệu

| # | File | Nội dung |
|---|---|---|
| 1 | [kafka_deployment_architecture.md](./kafka_deployment_architecture.md) | Distributed Log, ISR Replication, ZooKeeper → KRaft Evolution |
| 2 | [kafka_high_concurrency.md](./kafka_high_concurrency.md) | Exactly-Once Semantics, Idempotent Producer, Transactions, Rebalancing |
| 3 | [kafka_security.md](./kafka_security.md) | SASL Auth, ACLs, TLS/mTLS, Schema Registry |
| 4 | [kafka_subsystems.md](./kafka_subsystems.md) | Kafka Connect, Streams, ksqlDB, Event Sourcing, CQRS |

## Tổng Quan

```
📨 Apache Kafka
├── 🏗️ Infrastructure
│   ├── ZooKeeper → KRaft Consensus ─────── kafka_deployment_architecture.md
│   ├── Distributed Append-only Log ─────── kafka_deployment_architecture.md
│   └── ISR (In-Sync Replicas) ───────────── kafka_deployment_architecture.md
│
├── ⚡ Performance & Scalability
│   ├── Idempotent Producer (Deduplication) ── kafka_high_concurrency.md
│   ├── Transactional Exactly-Once (EOS) ───── kafka_high_concurrency.md
│   └── Cooperative Consumer Rebalancing ───── kafka_high_concurrency.md
│
├── 🔒 Security
│   ├── SASL/SCRAM & mTLS Authentication ────── kafka_security.md
│   ├── ACL Authorization ───────────────────── kafka_security.md
│   └── Schema Registry (Format validation) ─── kafka_security.md
│
└── 📦 Subsystems
    ├── Kafka Connect (Debezium CDC) ───────── kafka_subsystems.md
    ├── Kafka Streams & ksqlDB ─────────────── kafka_subsystems.md
    └── Event Sourcing + CQRS Patterns ─────── kafka_subsystems.md
```

## Key Patterns (Áp dụng cho NestJS)

| Pattern | Ý nghĩa hệ thống |
|---|---|
| **Event Sourcing** | Dùng Kafka làm Event Store thay vì chỉ lưu state hiện tại |
| **CQRS** | Tách biệt Write (commands) và Read (queries) |
| **Idempotency** | Xử lý duplicate messages an toàn với PID |
| **Debezium CDC** | Tự động đồng bộ DB (Postgres) ra Kafka real-time |
| **Schema validation** | Ngăn chặn data corruption với Avro/Protobuf |
