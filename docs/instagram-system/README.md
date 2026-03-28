# 📸 Instagram System - Phân Tích Kiến Trúc

> Tài liệu phân tích toàn diện hệ thống Instagram phục vụ 2+ tỷ users.

## Danh Sách Tài Liệu

| # | File | Nội dung |
|---|---|---|
| 1 | [instagram_deployment_system.md](./instagram_deployment_system.md) | CI/CD Pipeline, Canary Deploy, Rollback, Tech Stack |
| 2 | [instagram_system_architecture.md](./instagram_system_architecture.md) | 8 Sơ đồ kiến trúc: Upload, Feed, Deploy, Sharding, Caching, DM, Geo |
| 3 | [instagram_high_concurrency_analysis.md](./instagram_high_concurrency_analysis.md) | Hybrid Fan-out, Thundering Herd, Sharded Counters, Async Pipeline |
| 4 | [instagram_security_analysis.md](./instagram_security_analysis.md) | 8 lớp bảo mật: Network, Auth, API, Data, App, Content, Monitoring |
| 5 | [instagram_subsystems_analysis.md](./instagram_subsystems_analysis.md) | Media Pipeline, TAO, Data Pipeline, ML/AI, Search, Real-time, SRE |

## Tổng Quan Hệ Thống

```
📱 Instagram System
├── 🏗️ Infrastructure
│   ├── 1. Deployment (CI/CD) ─────────── instagram_deployment_system.md
│   └── 2. System Architecture ────────── instagram_system_architecture.md
│
├── ⚡ Performance
│   └── 3. High Concurrency ───────────── instagram_high_concurrency_analysis.md
│
├── 🔒 Security
│   └── 4. Security (8 layers) ────────── instagram_security_analysis.md
│
└── 📦 Subsystems
    ├── 5a. Media Processing Pipeline ─┐
    ├── 5b. Social Graph (TAO) ────────┤
    ├── 5c. Data Pipeline & Analytics ─┤
    ├── 5d. ML/AI Recommendation ──────┼── instagram_subsystems_analysis.md
    ├── 5e. Search & Discovery ────────┤
    ├── 5f. Real-time (DM, Live) ──────┤
    └── 5g. Reliability & SRE ─────────┘
```
