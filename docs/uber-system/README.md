# 🚗 Uber System - Phân Tích Kiến Trúc

> Tài liệu phân tích toàn diện hệ thống Uber phục vụ 130M+ users, 6.9B+ trips/năm.

## Danh Sách Tài Liệu

| # | File | Nội dung |
|---|---|---|
| 1 | [uber_deployment_architecture.md](./uber_deployment_architecture.md) | DOMA Architecture, Go/Java Stack, Deployment Pipeline |
| 2 | [uber_high_concurrency.md](./uber_high_concurrency.md) | H3 Geospatial, Ride Matching, Surge Pricing, ETA |
| 3 | [uber_security.md](./uber_security.md) | Mastermind Rules, RADAR Fraud, Payment Security, Safety |
| 4 | [uber_subsystems.md](./uber_subsystems.md) | Maps/Routing, Uber Eats, Michelangelo (ML), Data Pipeline |

## Tổng Quan

```
🚗 Uber System
├── 🏗️ Infrastructure
│   ├── DOMA (Domain-Oriented Architecture) ── uber_deployment_architecture.md
│   ├── Go + Java (high-perf & business) ──── uber_deployment_architecture.md
│   └── Peloton → Kubernetes ──────────────── uber_deployment_architecture.md
│
├── ⚡ Performance
│   ├── H3 Hexagonal Geospatial Index ─────── uber_high_concurrency.md
│   ├── Real-time Ride Matching ────────────── uber_high_concurrency.md
│   └── Surge Pricing (Kafka+Flink) ───────── uber_high_concurrency.md
│
├── 🔒 Security
│   ├── Mastermind + RADAR (Fraud) ─────────── uber_security.md
│   └── Physical Safety (SOS, RideCheck) ──── uber_security.md
│
└── 📦 Subsystems
    ├── Maps (GPS probes + routing) ────────── uber_subsystems.md
    ├── Uber Eats (3-sided marketplace) ────── uber_subsystems.md
    └── Michelangelo (ML Platform) ─────────── uber_subsystems.md
```

## Uber Unique Innovations

| Innovation | Significance |
|---|---|
| **H3** | Open-source hex grid — used by Foursquare, Databricks, DoorDash |
| **DOMA** | Domain-oriented architecture — influenced microservice design |
| **Jaeger** | Open-source distributed tracing — CNCF project |
| **Michelangelo** | End-to-end ML platform — influenced MLOps industry |
| **Apache Pinot** | Real-time analytics — sub-second on massive datasets |
