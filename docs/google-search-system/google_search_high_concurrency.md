# Google Search - Xử Lý Đồng Thời Cao & Search Pipeline

> 8.5B searches/ngày, < 200ms, fan-out to thousands of index shards.

---

## 1. Search Pipeline — End to End

```mermaid
sequenceDiagram
    actor User3 as 👤 User
    participant GFE2 as Google Front End
    participant QP as Query Processor
    participant Index as Index Servers (1000s)
    participant Ranker as Ranking (ML)
    participant KB as Knowledge Graph
    participant Ads as Ad Server

    User3->>GFE2: "best coffee shops near me"
    
    GFE2->>QP: Parse + understand query
    QP->>QP: Intent: local search
    QP->>QP: Entity: coffee shops
    QP->>QP: Expand: "cafe", "espresso bar"

    par Fan-out to index shards
        QP->>Index: Search shard 1..N (parallel)
    end

    Index-->>QP: Candidate documents (~10,000)

    QP->>Ranker: Stage 1: Fast scorer (BM25)
    Ranker-->>QP: Top 1,000
    
    QP->>Ranker: Stage 2: BERT/MUM (deep)
    Ranker-->>QP: Top 100

    QP->>Ranker: Stage 3: Final ranking
    Ranker-->>QP: Top 10

    par Enrich results
        QP->>KB: Knowledge panel?
        QP->>Ads: Relevant ads?
    end

    QP-->>GFE2: Results + Knowledge + Ads
    GFE2-->>User3: Search results (< 200ms)
```

---

## 2. Inverted Index — Core Data Structure

```mermaid
flowchart TB
    subgraph "Inverted Index"
        TERM1["'coffee'"] --> PL1["Doc1(title,3x), Doc5(body,7x), Doc12(h1,1x)"]
        TERM2["'shop'"] --> PL2["Doc1(title,2x), Doc3(body,4x), Doc12(body,5x)"]
        TERM3["'best'"] --> PL3["Doc1(title,1x), Doc5(title,1x), Doc99(body,2x)"]
    end

    subgraph "What each posting stores"
        POST["DocID, position, frequency,<br/>field (title/body/h1),<br/>term proximity"]
    end

    subgraph "Compression"
        COMP1["Delta encoding (DocIDs)"]
        COMP2["Variable-byte encoding"]
        COMP3["Skip lists (fast seeking)"]
    end

    style TERM1 fill:#4285F4,color:#fff
```

### Index Sharding

```mermaid
flowchart TB
    FULL_INDEX["📇 Full Index (~100B+ docs)"] --> SHARD_A["Shard A<br/>(Docs 1-1M)"]
    FULL_INDEX --> SHARD_B["Shard B<br/>(Docs 1M-2M)"]
    FULL_INDEX --> SHARD_N2["Shard N<br/>(Docs ...)"]

    QUERY6["Query"] --> FAN["Fan-out: query ALL shards in parallel"]
    SHARD_A & SHARD_B & SHARD_N2 --> MERGE2["Merge + global top-K ranking"]

    NOTE17["Each shard has replicas<br/>→ Route to least-loaded replica<br/>→ Tolerate shard failures"]

    style NOTE17 fill:#34A853,color:#fff
```

---

## 3. Crawler Architecture — Googlebot

```mermaid
flowchart TB
    SEED["🌱 Seed URLs"] --> FRONTIER2["📋 URL Frontier<br/>(Billions of URLs)"]

    FRONTIER2 --> PRIORITIZE["🏆 Prioritize:<br/>• PageRank (importance)<br/>• Change frequency<br/>• Freshness need"]
    PRIORITIZE --> POLITENESS["🤝 Politeness scheduler:<br/>• robots.txt check<br/>• Rate limit per domain<br/>• Crawl delay"]
    POLITENESS --> FETCH2["📥 Distributed Fetchers<br/>(Thousands of workers)"]

    FETCH2 --> PARSE2["🔧 Parse HTML:<br/>• Extract text<br/>• Extract links<br/>• Render JavaScript"]
    PARSE2 --> DEDUP2["🔍 Dedup:<br/>• URL normalization<br/>• SimHash (near-duplicate)"]
    DEDUP2 --> NEW_URLS["🔗 New URLs → Frontier"]
    DEDUP2 --> INDEX3["📇 Build inverted index"]

    style FRONTIER2 fill:#4285F4,color:#fff
    style POLITENESS fill:#FBBC04,color:#333
```

---

## 4. Multi-Stage Ranking Cascade

```mermaid
flowchart TB
    CANDIDATES["📚 100B+ docs in index"]
    
    CANDIDATES --> STAGE1["Stage 0: Boolean retrieval<br/>Inverted index match<br/>(~10,000 candidates, < 1ms)"]
    STAGE1 --> STAGE2["Stage 1: BM25 + features<br/>Fast statistical scoring<br/>(~1,000 candidates, < 5ms)"]
    STAGE2 --> STAGE3["Stage 2: BERT/MUM<br/>Deep semantic understanding<br/>(~100 candidates, < 50ms)"]
    STAGE3 --> STAGE4["Stage 3: Final blending<br/>Freshness, diversity, personalization<br/>(~10 results, < 10ms)"]
    STAGE4 --> PAGE["📄 Results page"]

    subgraph "Ranking Signals (1000+)"
        SIG_REL["📊 Relevance (BM25, BERT)"]
        SIG_AUTH["🏛️ Authority (PageRank)"]
        SIG_FRESH["⏰ Freshness"]
        SIG_UX["📱 Page experience (Core Web Vitals)"]
        SIG_LOCAL["📍 Location (for local intent)"]
        SIG_PERSON["👤 Personalization (search history)"]
    end

    style STAGE2 fill:#4285F4,color:#fff
    style STAGE3 fill:#EA4335,color:#fff
```

---

## 5. Latency Engineering — "Tail at Scale"

```mermaid
flowchart TB
    subgraph "The Problem"
        P1["Query fans out to 1000+ shards"]
        P2["Response time = slowest shard"]
        P3["P99 matters, not P50!"]
    end

    subgraph "Solutions"
        S1["🔄 Hedged requests:<br/>Send to 2 replicas, use faster response"]
        S2["⏰ Tied requests:<br/>First server to start cancels others"]
        S3["📊 Canary services:<br/>Test query on subset first"]
        S4["🚫 Good enough results:<br/>Timeout slow shards, return partial"]
        S5["🏗️ Micro-partitioning:<br/>More, smaller shards = less variance"]
    end

    style S1 fill:#34A853,color:#fff
```

---

## 6. So Sánh Search Patterns

| Pattern | Google Search | YouTube | Amazon | Spotify |
|---|---|---|---|---|
| **Index type** | Inverted index | Inverted + metadata | OpenSearch | Elasticsearch |
| **Ranking** | PageRank + BERT | Watch time + CTR | Conversion + relevance | Popularity + taste |
| **Corpus size** | 100B+ pages | 1B+ videos | 350M+ products | 100M+ tracks |
| **Latency target** | < 200ms | < 500ms | < 200ms | < 200ms |
| **Unique** | Knowledge Graph | Video transcripts | Sponsored rankings | Audio CNN features |

---

## Mapping → NestJS

| Pattern | Google | NestJS Implementation |
|---|---|---|
| **Inverted index** | Custom built | Elasticsearch / OpenSearch |
| **Fan-out query** | Parallel shard queries | `Promise.allSettled()` + timeout |
| **Multi-stage ranking** | BM25 → BERT | ES score + custom re-ranker |
| **Hedged requests** | 2 replicas, fastest wins | `Promise.race()` with 2 calls |
| **Web crawler** | Googlebot | `crawlee` + BullMQ scheduler |
| **URL frontier** | Priority queue | Redis sorted set (ZADD/ZPOPMIN) |
| **Deduplication** | SimHash | `simhash-js` npm |
