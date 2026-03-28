# YouTube - Subsystems Analysis

> Recommendation, Search, Comments, Shorts, Monetization, Reliability.

---

## 1. Recommendation Engine

```mermaid
flowchart TB
    CORPUS["🎬 Corpus: 800M+ videos"] --> STAGE_1

    subgraph "Stage 1: Candidate Generation (~1000)"
        STAGE_1["Two-Tower Model"]
        USER_T["👤 User Tower:<br/>• Watch history<br/>• Search history<br/>• Demographics<br/>• Subscriptions"]
        ITEM_T["🎬 Video Tower:<br/>• Video embeddings<br/>• Channel info<br/>• Tags/categories"]
        USER_T --> EMB1["User Embedding"]
        ITEM_T --> EMB2["Video Embedding<br/>(Pre-computed)"]
        EMB1 --> ANN2["ANN Search<br/>(ScaNN)"]
        EMB2 --> ANN2
        ANN2 --> CAND["~1000 candidates"]
    end

    CAND --> STAGE_2

    subgraph "Stage 2: Ranking"
        STAGE_2["Deep Neural Network"]
        FEATURES["Rich Features:<br/>• User-video interactions<br/>• Video freshness<br/>• Creator relationship<br/>• Context (time, device)"]
        OBJECTIVE["Optimize for:<br/>Expected Watch Time<br/>(NOT click-through rate!)"]
        STAGE_2 --> FEATURES --> OBJECTIVE
    end

    OBJECTIVE --> STAGE_3

    subgraph "Stage 3: Policy & Diversity"
        STAGE_3["Re-ranking"]
        RULES["Business Rules:<br/>• Diversity (not same channel)<br/>• Freshness boost<br/>• Remove policy violations<br/>• Ad insertion points"]
    end

    STAGE_3 --> FEED2["📱 Homepage / Up Next"]

    style OBJECTIVE fill:#e64980,color:#fff
```

### Watch Time vs CTR — Key Decision

| Metric | Problem |
|---|---|
| **Click-Through Rate** | Optimizes for clickbait → user dissatisfaction |
| **Watch Time** | Optimizes for engaging content → user retention ✅ |

**YouTube's insight:** Shift from CTR to Watch Time dramatically reduced clickbait and improved user satisfaction.

---

## 2. YouTube Shorts Architecture

```mermaid
flowchart TB
    SHORT["📱 Short video (< 60s)"] --> PIPELINE2["Same DAG pipeline<br/>(but optimized for vertical)"]
    PIPELINE2 --> ENCODE2["Encode: 9:16 aspect<br/>fewer quality levels"]
    ENCODE2 --> CDN6["CDN pre-cache<br/>(More aggressive for Shorts)"]

    subgraph "Shorts Feed"
        SWIPE["👆 Swipe-based feed"]
        PREFETCH["⚡ Prefetch next 3-5 Shorts"]
        INSTANT["Instant playback<br/>(< 200ms)"]
        LOOP["🔄 Auto-loop"]
    end

    subgraph "Recommendation Differences"
        TT_DIFF["Short-form signals:<br/>• Loop rate (key metric)<br/>• Swipe-away rate<br/>• Share rate<br/>• Comment rate"]
    end
```

---

## 3. Comments System

```mermaid
flowchart TB
    COMMENT["💬 User comments"] --> BIGTABLE3["Bigtable<br/>(video_id → comments)"]

    subgraph "Comment Ranking"
        RECENT["⏰ Most Recent"]
        TOP["👍 Top Comments<br/>(Likes - weighted by account age)"]
        CREATOR["📌 Creator pinned"]
    end

    subgraph "Moderation"
        AI_MOD2["🤖 AI Filter<br/>• Spam<br/>• Hate speech<br/>• Scam links"]
        HELD["⏸️ Held for Review<br/>(Creator setting)"]
        SHADOW3["👻 Shadow-hide<br/>(Known spammer)"]
    end

    COMMENT --> AI_MOD2
    AI_MOD2 --> BIGTABLE3

    style AI_MOD2 fill:#ffd43b,color:#333
```

---

## 4. Monetization Architecture

```mermaid
flowchart LR
    subgraph "Revenue Streams"
        ADS3["📺 Ads<br/>(Pre-roll, mid-roll, banners)"]
        PREMIUM["⭐ YouTube Premium<br/>(Subscription)"]
        SUPER2["💬 Super Chat / Stickers"]
        MEMBER["👑 Channel Memberships"]
        MERCH["🛍️ Merch Shelf"]
    end

    subgraph "Ad Serving Pipeline"
        BID["🏦 Ad Auction<br/>(Real-time bidding)"]
        TARGET["🎯 Targeting:<br/>• Demographics<br/>• Interests<br/>• Context<br/>• Remarketing"]
        FILL2["📊 Fill rate optimization"]
    end

    subgraph "Revenue Split"
        TOTAL["💰 Ad Revenue"] --> SPLIT["55% Creator<br/>45% YouTube"]
    end
```

---

## 5. Data Pipeline & Analytics

```mermaid
flowchart LR
    subgraph "Events (Trillions/day)"
        E_VIEW["Views"]
        E_SEARCH["Searches"]
        E_CLICK["Clicks"]
        E_AD["Ad impressions"]
    end

    E_VIEW & E_SEARCH & E_CLICK & E_AD --> PUBSUB4["📨 Pub/Sub"]

    PUBSUB4 --> DATAFLOW["⚡ Dataflow<br/>(Real-time stream)"]
    PUBSUB4 --> BIGQUERY["📊 BigQuery<br/>(Batch analytics)"]

    DATAFLOW --> REALTIME["Real-time:<br/>• Trending detection<br/>• Fraud alerts<br/>• Live viewer counts"]

    BIGQUERY --> REPORTS2["📈 YouTube Studio<br/>(Creator analytics)"]
    BIGQUERY --> ML4["🤖 ML Training<br/>(Recommendation models)"]
```

---

## 6. So Sánh Tổng Hợp: 5 Systems

| Dimension | YouTube | Netflix | Instagram | Twitter | WhatsApp |
|---|---|---|---|---|---|
| **Primary** | Video UGC | Video licensed | Photo social | Microblog | Messaging |
| **Language** | Python/C++/Java/Go | Java (Spring) | Python (Django) | Scala/Java | Erlang |
| **Database** | Vitess+Bigtable+Spanner | Cassandra+Aurora | PostgreSQL | Manhattan | Mnesia+MySQL |
| **CDN** | Google Edge | Open Connect | Meta CDN | Generic | N/A |
| **Search** | Custom + Elasticsearch | Elasticsearch | Unicorn | Earlybird | N/A |
| **Recommendation** | Two-Tower + Watch Time | Ensemble + everything | Two-Tower + engagement | Trending + ranking | N/A |
| **Content protection** | Content ID | DRM | AI moderation | Community Notes | E2EE |
| **Open source** | Vitess, VP9, AV1 | Netflix OSS | Less public | Zipkin, Finagle | Less public |

---

## YouTube Unique Innovations

| Innovation | Impact |
|---|---|
| **Vitess** | MySQL sharding at scale → now CNCF project (PlanetScale, Slack use it) |
| **VP9 / AV1** | Open video codecs → saved billions in bandwidth globally |
| **Content ID** | Automated copyright → $9B+ paid to rights holders |
| **Watch Time optimization** | Shifted industry from CTR to engagement metrics |
| **QUIC** | Created by Google → now HTTP/3 standard (RFC 9000) |
| **YouTube Shorts** | TikTok competitor → 70B+ daily views |

---

## Mapping → NestJS

| Subsystem | YouTube | NestJS Implementation |
|---|---|---|
| **Recommendation** | Two-Tower + TensorFlow | TensorFlow.js / Python via gRPC |
| **Shorts feed** | Prefetch + swipe | `@nestjs/websockets` + preload queue |
| **Comments** | Bigtable (video_id partitioned) | MongoDB / Cassandra (partitioned by video) |
| **Monetization** | Real-time ad auction | External ad network SDK |
| **Analytics** | Pub/Sub → BigQuery | Kafka → ClickHouse + Grafana |
| **Creator Studio** | BigQuery dashboards | Custom dashboard + ClickHouse queries |
