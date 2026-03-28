# 🛒 Amazon System - Phân Tích Kiến Trúc

> Tài liệu phân tích toàn diện hệ thống Amazon phục vụ 310M+ customers, 150M+ DDB req/s.

## Danh Sách Tài Liệu

| # | File | Nội dung |
|---|---|---|
| 1 | [amazon_deployment_architecture.md](./amazon_deployment_architecture.md) | SOA Pioneer, Bezos API Mandate, Two-Pizza Teams, AWS Stack |
| 2 | [amazon_high_concurrency.md](./amazon_high_concurrency.md) | Checkout Flow, DynamoDB, Chaotic Storage, Prime Day, Event-Driven |
| 3 | [amazon_security.md](./amazon_security.md) | PCI DSS, Fraud Detection, A-to-Z Guarantee, Data Protection |
| 4 | [amazon_subsystems.md](./amazon_subsystems.md) | Recommendation, Fulfillment Network, Advertising, GameDays |

## Tổng Quan

```
🛒 Amazon System
├── 🏗️ Infrastructure
│   ├── Bezos API Mandate (SOA origin) ────── amazon_deployment_architecture.md
│   ├── Two-Pizza Teams (org design) ──────── amazon_deployment_architecture.md
│   └── 100K+ deploys/day ─────────────────── amazon_deployment_architecture.md
│
├── ⚡ Performance
│   ├── DynamoDB (150M+ req/s) ────────────── amazon_high_concurrency.md
│   ├── Distributed Checkout (Saga) ───────── amazon_high_concurrency.md
│   └── Chaotic Storage (warehouse) ───────── amazon_high_concurrency.md
│
├── 🔒 Security
│   ├── PCI DSS Level 1 ──────────────────── amazon_security.md
│   └── A-to-Z Guarantee ─────────────────── amazon_security.md
│
└── 📦 Subsystems
    ├── Item-to-Item Recommendation ────────── amazon_subsystems.md
    ├── Multi-echelon Fulfillment ──────────── amazon_subsystems.md
    └── GameDays (Resilience Testing) ──────── amazon_subsystems.md
```

## Amazon Unique Innovations

| Innovation | Significance |
|---|---|
| **Bezos API Mandate** | Forced SOA → birthed AWS ($90B+ revenue) |
| **Two-Pizza Teams** | Influenced org design at Google, Spotify, etc. |
| **DynamoDB** | Industry-standard serverless NoSQL |
| **Chaotic Storage** | Counterintuitive but optimal warehouse design |
| **GameDays** | Proactive resilience testing → adopted industry-wide |
| **Cell-based Architecture** | Blast radius isolation pattern |
