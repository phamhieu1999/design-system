# 📺 YouTube System - Phân Tích Kiến Trúc

> Tài liệu phân tích toàn diện hệ thống YouTube phục vụ 2.5B+ users, 800M+ videos.

## Danh Sách Tài Liệu

| # | File | Nội dung |
|---|---|---|
| 1 | [youtube_deployment_architecture.md](./youtube_deployment_architecture.md) | Vitess, Bigtable, Spanner, Borg, Maglev, QUIC |
| 2 | [youtube_high_concurrency.md](./youtube_high_concurrency.md) | Encoding Pipeline, CDN, ABR, View Counting, Live Streaming |
| 3 | [youtube_security.md](./youtube_security.md) | Content ID, View Fraud Detection, Content Moderation |
| 4 | [youtube_subsystems.md](./youtube_subsystems.md) | Recommendation, Shorts, Comments, Monetization, Analytics |

## Tổng Quan

```
📺 YouTube System
├── 🏗️ Infrastructure
│   ├── Vitess (MySQL sharding) ─────────── youtube_deployment_architecture.md
│   ├── Google Infrastructure (Borg, Maglev)─ youtube_deployment_architecture.md
│   └── QUIC Protocol (HTTP/3) ─────────── youtube_deployment_architecture.md
│
├── ⚡ Performance
│   ├── Video Encoding Pipeline (AV1/VP9)── youtube_high_concurrency.md
│   ├── Google CDN + ISP Edge Cache ───── youtube_high_concurrency.md
│   └── View Count Validation ──────────── youtube_high_concurrency.md
│
├── 🔒 Security
│   ├── Content ID ($9B+ rights payments)── youtube_security.md
│   └── View Fraud Detection ──────────── youtube_security.md
│
└── 📦 Subsystems
    ├── Two-Tower Recommendation ────────── youtube_subsystems.md
    ├── YouTube Shorts (70B+ views/day) ── youtube_subsystems.md
    └── Monetization (Ad Auction) ──────── youtube_subsystems.md
```

## YouTube Unique Innovations

| Innovation | Significance |
|---|---|
| **Vitess** | MySQL sharding → CNCF project (PlanetScale, Slack use it) |
| **VP9 / AV1** | Open video codecs → saved billions in bandwidth globally |
| **Content ID** | Automated copyright → $9B+ paid to rights holders |
| **Watch Time** | Shifted industry from CTR to engagement metrics |
| **QUIC** | Created by Google → now HTTP/3 standard (RFC 9000) |
