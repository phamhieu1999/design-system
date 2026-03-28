# 🐦 Twitter/X System - Phân Tích Kiến Trúc

> Tài liệu phân tích toàn diện hệ thống Twitter/X phục vụ 500M+ users, 500K tweets/phút.

## Danh Sách Tài Liệu

| # | File | Nội dung |
|---|---|---|
| 1 | [twitter_deployment_architecture.md](./twitter_deployment_architecture.md) | Tech Stack (Scala/Finagle), Snowflake ID, Infra Evolution (Mesos→K8s→GCP) |
| 2 | [twitter_high_concurrency.md](./twitter_high_concurrency.md) | Timeline Fan-out (Hybrid Push/Pull), Trending Topics, Engagement Counters |
| 3 | [twitter_security.md](./twitter_security.md) | 8 lớp bảo mật, DDoS incidents, Bot Detection, Anti-Scraping |
| 4 | [twitter_subsystems.md](./twitter_subsystems.md) | Earlybird Search, Manhattan Storage, Streaming API, Data Pipeline, SRE |

## Tổng Quan Hệ Thống

```
🐦 Twitter/X System
├── 🏗️ Infrastructure
│   ├── Tech Stack (Scala, Finagle, JVM) ─────── twitter_deployment_architecture.md
│   ├── Snowflake ID Generator ────────────────── twitter_deployment_architecture.md
│   └── Infra Evolution (Mesos → K8s → GCP) ──── twitter_deployment_architecture.md
│
├── ⚡ Performance
│   ├── Timeline Fan-out (Push/Pull) ──────────── twitter_high_concurrency.md
│   ├── Trending Topics Detection ─────────────── twitter_high_concurrency.md
│   └── Engagement Counters ───────────────────── twitter_high_concurrency.md
│
└── 📦 Subsystems
    ├── Earlybird (Real-time Search) ──────────── twitter_subsystems.md
    ├── Manhattan (KV Store) ──────────────────── twitter_subsystems.md
    ├── Streaming API (Firehose) ──────────────── twitter_subsystems.md
    ├── Data Pipeline (Heron/Spark) ───────────── twitter_subsystems.md
    └── Reliability (Zipkin, Circuit Breaker) ─── twitter_subsystems.md
```

## Twitter Unique Innovations

| Innovation | Significance |
|---|---|
| **Snowflake** | Industry standard distributed ID — used by Discord, Instagram variants |
| **Finagle** | Pioneered async RPC for microservices |
| **Zipkin** | De-facto distributed tracing standard |
| **Heron** | 10x throughput replacement for Apache Storm |
| **Manhattan** | Multi-tenant KV store replacing Cassandra + MySQL |
