# URL Shortener - Security Analysis

> URL shorteners are prime targets for phishing, spam, malware distribution.

---

## Tổng Quan

```mermaid
graph TB
    subgraph "Layer 1: Input Validation"
        URL_VAL["🔍 URL Validation"]
        MALWARE["🦠 Malware/Phishing check"]
    end

    subgraph "Layer 2: Abuse Prevention"
        RATE4["🚦 Rate Limiting"]
        SPAM4["🗑️ Spam Detection"]
        CAPTCHA2["🤖 CAPTCHA"]
    end

    subgraph "Layer 3: Link Safety"
        SAFE_BROWSE["🛡️ Google Safe Browsing"]
        BLACKLIST2["📋 URL Blacklist"]
        PREVIEW["👁️ Link Preview"]
    end

    subgraph "Layer 4: Access Control"
        API_AUTH2["🔑 API Key Authentication"]
        LINK_PWD["🔒 Password-protected links"]
        EXPIRY2["⏰ Link expiration"]
    end
```

---

## 1. Malicious URL Detection

```mermaid
flowchart TB
    URL2["📥 Long URL submitted"] --> VALIDATE["✅ URL format validation<br/>(Protocol, domain, encoding)"]
    VALIDATE --> SAFE["🛡️ Google Safe Browsing API<br/>(Phishing, malware, unwanted software)"]
    SAFE --> BLACKLIST3["📋 Custom blacklist<br/>(Known bad domains)"]
    BLACKLIST3 --> ML12["🤖 ML Classifier<br/>(Suspicious patterns)"]

    ML12 --> RESULT{"Safe?"}
    RESULT -->|"Yes"| CREATE2["✅ Create short URL"]
    RESULT -->|"No"| BLOCK7["🚫 Block + report"]
    RESULT -->|"Suspicious"| FLAG["⚠️ Flag for review"]

    subgraph "Suspicious Patterns"
        PAT1["• Homograph attacks (paypaI.com)"]
        PAT2["• URL encoding tricks"]
        PAT3["• Open redirect chains"]
        PAT4["• Known phishing kits"]
    end
```

---

## 2. Rate Limiting

```mermaid
flowchart TB
    subgraph "Token Bucket Algorithm"
        BUCKET["🪣 Bucket (capacity: 100)"]
        REFILL["Refill: 10 tokens/second"]
        REQ11["Request → consume 1 token"]
        EMPTY["Empty? → 429 Too Many Requests"]
    end

    subgraph "Tiers"
        ANON["Anonymous: 10/hour"]
        FREE2["Free API: 100/hour"]
        PAID["Paid API: 10,000/hour"]
        ENTERPRISE2["Enterprise: Unlimited"]
    end
```

---

## 3. Click-time Safety

```mermaid
sequenceDiagram
    actor User2 as 👤 User clicks short URL
    participant Service as Redirect Service
    participant Safe as Safe Browsing

    User2->>Service: GET /abc123
    Service->>Service: Lookup long URL
    
    alt URL was checked on creation
        Service->>Service: Check last_scan_date
        alt Scan < 24h ago
            Service-->>User2: 302 redirect
        else Scan > 24h ago
            Service->>Safe: Re-check URL
            alt Still safe
                Service-->>User2: 302 redirect
            else Now flagged
                Service-->>User2: ⚠️ Warning interstitial page
            end
        end
    end
```

---

## 4. So Sánh Security: URL Shortener vs Others

| Layer | URL Shortener | Stripe | Amazon | WhatsApp |
|---|---|---|---|---|
| **Primary threat** | Phishing/malware | Payment fraud | Marketplace fraud | Surveillance |
| **Detection** | Safe Browsing + ML | Radar network ML | A-to-Z + seller verification | E2EE |
| **Rate limiting** | Per-IP / per-key | Per-API-key | Per-account | Connection-level |
| **Unique** | Interstitial warnings | Idempotency keys | Chaotic storage | Signal Protocol |

---

## Mapping → NestJS

| Pattern | NestJS Implementation |
|---|---|
| **Safe Browsing** | Google Safe Browsing API v4 |
| **URL validation** | `class-validator` + custom pipe |
| **Rate limiting** | `@nestjs/throttler` (per IP + per key) |
| **Blacklist** | Redis SET + `SISMEMBER` |
| **ML classifier** | Python microservice via gRPC |
| **Link expiration** | PostgreSQL TTL + Redis TTL |
| **Password links** | `bcrypt` hash in DB |
