# URL Shortener - Xử Lý Đồng Thời Cao & Scalability

> 100:1 read-write ratio, < 10ms redirect latency, billions of URLs.

---

## 1. Caching Strategy — Core to Performance

```mermaid
flowchart TB
    subgraph "Multi-Layer Cache"
        L1["L1: CDN Edge Cache<br/>(Viral links, < 1ms)"]
        L2["L2: Redis/Memcached<br/>(All hot links, < 2ms)"]
        L3["L3: Database<br/>(All links, 5-20ms)"]
    end

    REQ9["Request"] --> L1
    L1 -->|MISS| L2
    L2 -->|MISS| L3
    L3 --> POPULATE["Populate L2 cache"]

    subgraph "Cache Policies"
        LRU["LRU eviction (80/20 rule:<br/>20% URLs get 80% traffic)"]
        TTL2["TTL: 24-48 hours"]
        WARM["Cache warming: Pre-load popular URLs"]
    end

    style L1 fill:#1DB954,color:#fff
```

### Cache Hit Rate Impact

| Cache Hit Rate | DB Queries/s (at 100K req/s) | Latency |
|---|---|---|
| **0% (no cache)** | 100,000 | 10-20ms |
| **80%** | 20,000 | ~3ms avg |
| **95%** | 5,000 | ~2ms avg |
| **99%** | 1,000 | ~1ms avg |

---

## 2. Database Sharding

```mermaid
flowchart TB
    REQ10["short.ly/abc123"] --> HASH5["Hash: abc123 % N shards"]
    HASH5 --> SHARD{"Consistent Hashing Ring"}

    SHARD --> S1["Shard 1<br/>(a-f → 300M URLs)"]
    SHARD --> S2["Shard 2<br/>(g-m → 300M URLs)"]
    SHARD --> S3["Shard 3<br/>(n-s → 300M URLs)"]
    SHARD --> S4["Shard 4<br/>(t-z → 300M URLs)"]

    NOTE16["Consistent hashing:<br/>Add/remove shard →<br/>only move ~1/N data"]

    style NOTE16 fill:#4c6ef5,color:#fff
```

### Sharding Strategies

| Strategy | Pros | Cons |
|---|---|---|
| **Hash-based** | Even distribution | Cross-shard queries hard |
| **Range-based** | Easy range queries | Hot spots possible |
| **Consistent hashing** | Minimal data movement | More complex |

---

## 3. 301 vs 302 Redirect

```mermaid
flowchart TB
    subgraph "301 Permanent Redirect"
        R301["Browser caches redirect"]
        R301 --> PRO301["✅ Faster (no server hit)"]
        R301 --> CON301["❌ Can't track clicks<br/>❌ Can't change destination"]
    end

    subgraph "302 Temporary Redirect (Recommended)"
        R302["Browser always hits server"]
        R302 --> PRO302["✅ Track every click<br/>✅ Can update destination"]
        R302 --> CON302["❌ Higher server load"]
    end

    style R302 fill:#51cf66,color:#fff
```

---

## 4. Analytics Pipeline

```mermaid
flowchart LR
    CLICK2["🖱️ Click"] --> REDIRECT2["302 Redirect<br/>(< 5ms)"]
    REDIRECT2 --> KAFKA23["📨 Kafka<br/>(Async fire-and-forget)"]

    KAFKA23 --> ENRICH["🔧 Enrich Worker:<br/>• GeoIP → country<br/>• User-Agent → device/OS<br/>• Referer → source"]
    ENRICH --> CLICKHOUSE["📊 ClickHouse<br/>(Columnar analytics DB)"]

    CLICKHOUSE --> DASHBOARD["📈 Dashboard:<br/>• Clicks by time<br/>• Geographic distribution<br/>• Device breakdown<br/>• Top referrers"]
```

---

## 5. Handling Viral/Hot URLs

```mermaid
flowchart TB
    VIRAL["🔥 Viral URL<br/>(1M+ clicks/hour)"] --> DETECT3["Detect: Counter exceeds threshold"]
    DETECT3 --> PROMOTE["Promote to CDN cache<br/>(Global edge, < 1ms)"]
    PROMOTE --> SCALE2["Scale read replicas<br/>(Auto-scaling)"]

    subgraph "Protection"
        BLOOM["Bloom Filter:<br/>Quick check 'does URL exist?'<br/>(Avoid DB for 404s)"]
        RATE3["Rate limiting per IP<br/>(Prevent DDoS via short URL)"]
    end

    style VIRAL fill:#ff6b6b,color:#fff
```

---

## 6. Availability — Multi-Region

```mermaid
graph TB
    subgraph "Region US-East"
        US_LB["⚖️ LB"]
        US_APP["App (3 replicas)"]
        US_CACHE["Redis"]
        US_DB["DB (Primary)"]
    end

    subgraph "Region EU-West"
        EU_LB["⚖️ LB"]
        EU_APP["App (3 replicas)"]
        EU_CACHE["Redis"]
        EU_DB["DB (Replica)"]
    end

    DNS["🌍 GeoDNS"] --> US_LB & EU_LB
    US_DB -- "Async replication" --> EU_DB
```

---

## Mapping → NestJS

| Pattern | NestJS Implementation |
|---|---|
| **Multi-layer cache** | CDN (CloudFront) + `@nestjs/cache-manager` + Redis |
| **Sharding** | Consistent hash ring + multiple DB connections |
| **Bloom filter** | `bloom-filters` npm in Redis |
| **Analytics pipeline** | Kafka producer (async) → ClickHouse |
| **Hot URL detection** | Redis counter + threshold alert |
| **Rate limiting** | `@nestjs/throttler` per IP |
| **302 redirect** | `@Redirect()` decorator or `res.redirect(302)` |
