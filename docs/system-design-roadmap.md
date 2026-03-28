# 🗺️ Roadmap System Design - Học Từ Hệ Thống Thực Tế

> Mục tiêu: Phân tích kiến trúc các hệ thống lớn để học design patterns áp dụng cho NestJS microservices.

---

## Tier 1: Social & Real-time (Nền Tảng)

- [x] **Instagram** — Fan-out, media pipeline, sharding, ML ranking
- [x] **Twitter/X** — Timeline fan-out, trending, pub/sub, event sourcing
- [x] **WhatsApp** — E2EE messaging, delivery semantics, offline sync
- [ ] **URL Shortener** — Hashing, base62, read-heavy, redirect optimization

## Tier 2: Media & Streaming

- [x] **YouTube** — Video transcoding, adaptive bitrate, recommendation
- [x] **Netflix** — Microservices gốc, chaos engineering, global CDN
- [ ] **Spotify** — Audio streaming, collaborative filtering, offline mode

## Tier 3: E-commerce & Fintech

- [x] **Amazon** — Distributed checkout, inventory, warehouse management
- [x] **Stripe/Payment** — Idempotency, distributed transactions, PCI
- [x] **Uber** — Geo-spatial indexing, matching, surge pricing, real-time tracking

## Tier 4: Infrastructure & Database

- [ ] **Google Search** — Inverted index, PageRank, web crawler
- [ ] **Kafka** — Distributed log, partitioning, exactly-once delivery
- [ ] **Redis** — In-memory structures, persistence, cluster mode

---

## Pattern Coverage Matrix

```
Pattern             T1                    T2              T3              T4
                    IG  TW  WA  URL     YT  NF  SP     AM  ST  UB     GS  KF  RD
─────────────────────────────────────────────────────────────────────────────────
Fan-out             ✅  ✅                                              
Sharding            ✅  ✅  ✅          ✅  ✅          ✅      ✅          ✅
Real-time           ✅      ✅                          ✅      ✅
Video/Media         ✅                  ✅  ✅  ✅
ML/Recommendation   ✅  ✅              ✅  ✅  ✅       ✅
Geo-spatial                                                     ✅
Transactions                                            ✅  ✅
Chaos Engineering                           ✅
Event Sourcing          ✅  ✅                                   ✅
Rate Limiting       ✅  ✅                   ✅
Idempotency                 ✅                           ✅  ✅
Caching             ✅  ✅  ✅          ✅  ✅          ✅              ✅
```

## Thứ Tự Recommended

| # | Hệ thống | Lý do ưu tiên |
|---|---|---|
| 1 | ~~Instagram~~ ✅ | Done — full-stack social platform |
| 2 | Twitter/X | Gần Instagram, học thêm trending + event sourcing |
| 3 | WhatsApp | Real-time messaging sâu, E2E encryption |
| 4 | Netflix | Microservices patterns, chaos engineering |
| 5 | Uber | Geo-spatial + matching — pattern hoàn toàn mới |
| 6 | YouTube | Video pipeline nâng cao |
| 7 | Amazon | E-commerce distributed systems |
| 8 | Kafka | Hiểu sâu messaging infrastructure |
