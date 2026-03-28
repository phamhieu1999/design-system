# 🔗 URL Shortener System - Phân Tích Kiến Trúc

> Classic system design — Bitly xử lý 28B+ clicks/tháng, < 10ms redirect.

## Danh Sách Tài Liệu

| # | File | Nội dung |
|---|---|---|
| 1 | [url_shortener_deployment_architecture.md](./url_shortener_deployment_architecture.md) | Core Design, ID Generation (Base62/Snowflake), Data Model |
| 2 | [url_shortener_high_concurrency.md](./url_shortener_high_concurrency.md) | Multi-Layer Cache, Sharding, 301/302, Analytics Pipeline |
| 3 | [url_shortener_security.md](./url_shortener_security.md) | Malicious URL Detection, Rate Limiting, Safe Browsing |
| 4 | [url_shortener_subsystems.md](./url_shortener_subsystems.md) | Analytics Dashboard, Custom Domains, QR Codes, API Design |

## Tổng Quan

```
🔗 URL Shortener System
├── 🏗️ Infrastructure
│   ├── Base62/Snowflake ID Generation ─────── url_shortener_deployment_architecture.md
│   ├── NoSQL (Cassandra/DynamoDB) ─────────── url_shortener_deployment_architecture.md
│   └── Read-Heavy (100:1 ratio) ───────────── url_shortener_deployment_architecture.md
│
├── ⚡ Performance
│   ├── Multi-Layer Cache (CDN+Redis+DB) ───── url_shortener_high_concurrency.md
│   ├── Consistent Hashing (Sharding) ──────── url_shortener_high_concurrency.md
│   └── Async Analytics (Kafka→ClickHouse) ── url_shortener_high_concurrency.md
│
├── 🔒 Security
│   ├── Google Safe Browsing API ────────────── url_shortener_security.md
│   └── Token Bucket Rate Limiting ──────────── url_shortener_security.md
│
└── 📦 Subsystems
    ├── Real-time Analytics Dashboard ────────── url_shortener_subsystems.md
    └── Custom Domains + QR Generation ───────── url_shortener_subsystems.md
```

## Key Patterns (Reusable Across Systems)

| Pattern | URL Shortener | Also seen in |
|---|---|---|
| **Snowflake ID** | Collision-free IDs | Twitter, Instagram |
| **Consistent Hashing** | Even data distribution | Cassandra, DynamoDB |
| **Bloom Filter** | Fast existence check | Bigtable, Medium |
| **Token Bucket** | Rate limiting | Stripe, All APIs |
| **302 Redirect** | Trackable redirects | All shorteners |
