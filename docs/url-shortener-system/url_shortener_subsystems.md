# URL Shortener - Subsystems Analysis

> Analytics, Custom Domains, QR Codes, Link Management, API Design.

---

## 1. Analytics Dashboard

```mermaid
flowchart TB
    CLICKS["🖱️ Click events<br/>(Kafka stream)"]

    CLICKS --> METRICS

    subgraph "Real-time Metrics"
        M1["📊 Clicks over time<br/>(By hour, day, week)"]
        M2["🌍 Geographic distribution<br/>(Country, city)"]
        M3["📱 Device breakdown<br/>(Mobile, Desktop, Tablet)"]
        M4["🌐 Browser & OS"]
        M5["🔗 Top referrers"]
        M6["🕐 Peak hours"]
    end

    subgraph "Storage"
        CLICKHOUSE2["📊 ClickHouse<br/>(Columnar, fast aggregation)"]
        TIMESERIES2["📈 Pre-aggregated counters<br/>(Redis sorted sets)"]
    end

    METRICS --> CLICKHOUSE2 & TIMESERIES2
```

---

## 2. Custom Short Domains

```mermaid
flowchart TB
    BRAND["🏢 Brand: nike.link/shoes"] --> DNS2["DNS CNAME → shortener service"]
    DNS2 --> CERT2["🔐 Auto SSL (Let's Encrypt)"]
    CERT2 --> ROUTE2["🔀 Route by domain<br/>→ Brand's link pool"]

    subgraph "Multi-tenant"
        TENANT1["nike.link → Nike's links"]
        TENANT2["ibm.co → IBM's links"]
        TENANT3["bit.ly → Public links"]
    end
```

---

## 3. QR Code Generation

```mermaid
flowchart LR
    SHORT3["short.ly/abc123"] --> QR["📱 QR Code Generator<br/>(Server-side rendering)"]
    QR --> CUSTOMIZE["Customize:<br/>• Brand colors<br/>• Logo overlay<br/>• Size/format (PNG, SVG)"]
    CUSTOMIZE --> CDN_CACHE["📦 Cache on CDN"]
```

---

## 4. Link Management Features

| Feature | Description |
|---|---|
| **Custom aliases** | `short.ly/my-brand-campaign` |
| **Link expiration** | Auto-expire after date or N clicks |
| **Link editing** | Change destination URL after creation |
| **UTM builder** | Auto-append UTM parameters |
| **A/B testing** | Split traffic between destinations |
| **Deep links** | Mobile app deep linking |
| **Bulk operations** | CSV import/export |
| **Tags & folders** | Organize links by campaign |

---

## 5. API Design (Bitly-style)

```mermaid
flowchart TB
    subgraph "REST API"
        CREATE3["POST /v4/shorten<br/>{long_url, domain, custom}"]
        GET2["GET /v4/bitlinks/{id}"]
        CLICKS2["GET /v4/bitlinks/{id}/clicks<br/>?unit=day&units=30"]
        UPDATE2["PATCH /v4/bitlinks/{id}"]
        DELETE2["DELETE /v4/bitlinks/{id}"]
    end

    subgraph "API Features"
        AUTH7["🔑 OAuth 2.0 + API tokens"]
        RATE5["🚦 Rate limiting (per token)"]
        WEBHOOK3["📨 Webhooks (click milestones)"]
        BATCH["📦 Batch operations"]
    end
```

---

## 6. Key Design Decisions Summary

| Decision | Choice | Rationale |
|---|---|---|
| **ID generation** | Snowflake → Base62 | Collision-free, distributed |
| **Database** | Cassandra / DynamoDB | Key-value, horizontal scale |
| **Cache** | Redis (LRU) | 80/20 rule, sub-ms lookups |
| **Redirect type** | 302 (Temporary) | Click tracking required |
| **Analytics** | Async (Kafka → ClickHouse) | Don't slow down redirects |
| **Availability** | Multi-region active-active | Global users, low latency |
| **API style** | REST + versioned | Developer-friendly |

---

## So Sánh: URL Shortener Design Patterns Applied

| Pattern | URL Shortener | Also Used In |
|---|---|---|
| **Base62/Snowflake** | ID generation | Twitter (Snowflake), Instagram |
| **Consistent hashing** | DB sharding | Cassandra, DynamoDB, Discord |
| **Bloom filter** | Existence check | Google Bigtable, Medium |
| **CDN caching** | Hot URL redirect | Netflix, YouTube, Spotify |
| **Async analytics** | Kafka → ClickHouse | Uber, Spotify, Instagram |
| **Rate limiting** | Token bucket | Stripe, GitHub, All APIs |
| **302 redirect** | Trackable redirect | All URL shorteners |

---

## Mapping → NestJS (Full Implementation)

| Component | NestJS Implementation |
|---|---|
| **Shorten API** | `@Post('shorten')` + DTO validation |
| **Redirect** | `@Get(':code')` + `@Redirect()` |
| **ID generation** | `snowflake-id` + custom Base62 |
| **Cache** | `CacheModule` + Redis |
| **Database** | TypeORM + PostgreSQL (or DynamoDB) |
| **Analytics** | Kafka producer → consumer → ClickHouse |
| **Rate limiting** | `@nestjs/throttler` |
| **Auth** | `@nestjs/passport` + API key strategy |
| **QR codes** | `qrcode` npm package |
| **Custom domains** | Multi-tenant guard + domain lookup |
