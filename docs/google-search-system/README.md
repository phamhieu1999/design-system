# 🔍 Google Search System - Phân Tích Kiến Trúc

> Tài liệu phân tích Google Search — 8.5B+ searches/ngày, < 200ms, hundreds of billions pages indexed.

## Danh Sách Tài Liệu

| # | File | Nội dung |
|---|---|---|
| 1 | [google_search_deployment_architecture.md](./google_search_deployment_architecture.md) | Colossus, Bigtable, Spanner, TrueTime, Borg |
| 2 | [google_search_high_concurrency.md](./google_search_high_concurrency.md) | Search Pipeline, Inverted Index, Googlebot, Multi-stage Ranking |
| 3 | [google_search_security.md](./google_search_security.md) | SpamBrain, Safe Browsing, E-E-A-T |
| 4 | [google_search_subsystems.md](./google_search_subsystems.md) | PageRank, Knowledge Graph, Ads Auction, AI Evolution |

## Tổng Quan

```
🔍 Google Search
├── 🏗️ Infrastructure
│   ├── Colossus (Distributed FS, exabytes) ── google_search_deployment_architecture.md
│   ├── Bigtable (NoSQL, petabytes) ─────────── google_search_deployment_architecture.md
│   └── Spanner + TrueTime (global ACID) ────── google_search_deployment_architecture.md
│
├── ⚡ Performance
│   ├── Inverted Index (100B+ docs) ─────────── google_search_high_concurrency.md
│   ├── Multi-stage Ranking Cascade ─────────── google_search_high_concurrency.md
│   └── Tail-at-Scale (hedged requests) ─────── google_search_high_concurrency.md
│
├── 🔒 Security
│   ├── SpamBrain (neural spam detector) ────── google_search_security.md
│   └── Safe Browsing (4B+ devices) ─────────── google_search_security.md
│
└── 📦 Subsystems
    ├── PageRank (graph authority) ───────────── google_search_subsystems.md
    ├── Knowledge Graph (5B+ entities) ──────── google_search_subsystems.md
    └── BERT / MUM / Gemini (AI ranking) ────── google_search_subsystems.md
```

## Google Search Unique Innovations

| Innovation | Significance |
|---|---|
| **PageRank** | Graph-based authority → transformed web search |
| **MapReduce** | Distributed batch processing → inspired Hadoop |
| **GFS → Colossus** | Distributed FS → inspired HDFS |
| **Bigtable** | Wide-column NoSQL → inspired HBase, Cassandra |
| **Spanner + TrueTime** | Globally consistent DB with atomic clocks |
| **Safe Browsing** | Privacy-preserving malware detection → 4B devices |
