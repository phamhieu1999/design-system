# 🎬 Netflix System - Phân Tích Kiến Trúc

> Tài liệu phân tích toàn diện hệ thống Netflix phục vụ 260M+ subscribers tại 190+ quốc gia.

## Danh Sách Tài Liệu

| # | File | Nội dung |
|---|---|---|
| 1 | [netflix_deployment_architecture.md](./netflix_deployment_architecture.md) | Tech Stack (Java/Spring), Netflix OSS, Spinnaker CD, Titus |
| 2 | [netflix_high_concurrency.md](./netflix_high_concurrency.md) | Encoding Pipeline, Open Connect CDN, ABR, Chaos Engineering |
| 3 | [netflix_security.md](./netflix_security.md) | DRM Multi-platform, Forensic Watermarking, Device Management |
| 4 | [netflix_subsystems.md](./netflix_subsystems.md) | Recommendation, A/B Testing, Search, Data Pipeline, Observability |

## Tổng Quan Hệ Thống

```
🎬 Netflix System
├── 🏗️ Infrastructure
│   ├── Java/Spring Boot + Virtual Threads ─── netflix_deployment_architecture.md
│   ├── Netflix OSS (Eureka, Zuul, Spinnaker)─ netflix_deployment_architecture.md
│   └── Titus Container Platform ──────────── netflix_deployment_architecture.md
│
├── ⚡ Performance
│   ├── Per-title Encoding Pipeline ────────── netflix_high_concurrency.md
│   ├── Open Connect CDN (inside ISPs!) ────── netflix_high_concurrency.md
│   └── Chaos Engineering (Simian Army) ────── netflix_high_concurrency.md
│
├── 🔒 Security
│   ├── DRM (Widevine/FairPlay/PlayReady) ──── netflix_security.md
│   └── Forensic Watermarking ──────────────── netflix_security.md
│
└── 📦 Subsystems
    ├── Recommendation Engine ──────────────── netflix_subsystems.md
    ├── A/B Testing (250+ tests) ───────────── netflix_subsystems.md
    └── Real-time Data Pipeline ────────────── netflix_subsystems.md
```

## Netflix Unique Innovations

| Innovation | Significance |
|---|---|
| **Netflix OSS** | Pioneered entire microservices ecosystem (Spring Cloud) |
| **Chaos Engineering** | "Test in production" — adopted by Amazon, Google, Microsoft |
| **Open Connect** | CDN inside ISPs (free!) — unique model, no other company does this |
| **Per-title Encoding** | 20% bandwidth savings — adopted by YouTube, Disney+ |
| **VMAF** | Open-source perceptual quality metric — now industry standard |
| **Spinnaker** | Multi-region CD — CNCF project |
