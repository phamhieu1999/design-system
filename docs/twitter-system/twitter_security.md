# Twitter/X - Security Analysis

> Phân tích 8 lớp bảo mật của Twitter/X và các bài học từ security incidents thực tế.

---

## Tổng Quan: Defense in Depth

```mermaid
graph TB
    subgraph "Layer 1: Network Edge"
        DDOS["🌍 DDoS Mitigation<br/>(Anycast + 3rd-party)"]
        WAF2["🛡️ WAF + CAPTCHA"]
        TLS2["🔒 TLS 1.3"]
    end

    subgraph "Layer 2: Authentication"
        OAUTH3["🔑 OAuth 1.0a / 2.0"]
        MFA2["📱 2FA (TOTP, SMS, Security Key)"]
        SESSION2["🎫 Session + Device Tracking"]
    end

    subgraph "Layer 3: API Security"
        RL2["⚖️ Rate Limiting (Tiered)"]
        SCOPE["🔐 Scoped Access (App Review)"]
        SCRAPE["🚫 Anti-Scraping"]
    end

    subgraph "Layer 4: Application"
        BOT["🤖 Bot Detection (Behavioral)"]
        SPAM["🗑️ Spam Prevention"]
        CONTENT2["📋 Content Policy AI"]
    end

    subgraph "Layer 5: Data"
        ENC["🔐 Encryption at Rest"]
        PII["👤 PII Protection"]
        AUDIT2["📋 Audit Trail"]
    end

    subgraph "Layer 6: Incident Response"
        MONITOR["👁️ Real-time Monitoring"]
        IRT["🚨 Incident Response Team"]
        DISC["📢 Disclosure Process"]
    end

    DDOS --> WAF2 --> TLS2 --> OAUTH3 --> MFA2 --> SESSION2 --> RL2 --> SCOPE --> SCRAPE --> BOT --> SPAM --> CONTENT2 --> ENC --> PII --> AUDIT2 --> MONITOR --> IRT --> DISC

    style DDOS fill:#4c6ef5,color:#fff
    style OAUTH3 fill:#7950f2,color:#fff
    style RL2 fill:#ae3ec9,color:#fff
    style BOT fill:#e64980,color:#fff
    style ENC fill:#ff922b,color:#fff
    style MONITOR fill:#ff6b6b,color:#fff
```

---

## 1. Network Security & DDoS Protection

```mermaid
flowchart TB
    ATTACK2["🔴 DDoS Attack"] --> LAYER1["Layer 1: Anycast<br/>Distribute across PoPs"]
    LAYER1 --> LAYER2_N["Layer 2: Traffic Analysis<br/>• IP reputation scoring<br/>• Geo anomaly detection<br/>• Rate-based filtering"]
    LAYER2_N --> LAYER3_N["Layer 3: CAPTCHA Challenge<br/>(Emergency measure)"]
    LAYER3_N --> LAYER4_N["Layer 4: Application WAF<br/>• SQL injection<br/>• XSS filtering<br/>• Bot signatures"]
    LAYER4_N --> ORIGIN3["✅ Origin Servers"]

    style ATTACK2 fill:#ff6b6b,color:#fff
    style ORIGIN3 fill:#51cf66,color:#fff
```

### DDoS Incidents (Bài Học Thực Tế)

| Năm | Incident | Impact | Response |
|---|---|---|---|
| 2016 | Mirai botnet → Dyn DNS | Twitter down toàn cầu | Diversify DNS providers |
| 2024 | Hacktivist DDoS campaigns | Intermittent outages | 3rd-party DDoS protection |
| 2025 | Large-scale volumetric attack | Service degradation | CAPTCHA + emergency rate limits |

**Bài học:** Không chỉ protect origin, phải protect cả DNS và CDN layer.

---

## 2. Authentication — OAuth 1.0a & 2.0

```mermaid
sequenceDiagram
    actor User as 👤 User
    participant App as 🏢 3rd Party App
    participant X as 🔑 X Auth Server
    participant API as 📡 X API

    Note over App,X: OAuth 2.0 with PKCE (recommended)

    App->>X: GET /oauth2/authorize?code_challenge=xxx
    X->>User: Login + consent screen
    User->>X: Approve scopes
    X->>App: authorization_code

    App->>X: POST /oauth2/token<br/>(code + code_verifier)
    X-->>App: access_token + refresh_token

    App->>API: GET /tweets?access_token=xxx
    API-->>App: Tweet data (scoped)

    Note over App,API: Token scopes:<br/>tweet.read, tweet.write,<br/>users.read, dm.read, etc.
```

### Access Tiers

| Tier | Rate Limit | Cost | Use Case |
|---|---|---|---|
| **Free** | 1,500 tweets/month read | Free | Hobby projects |
| **Basic** | 10K tweets/month | $100/month | Small apps |
| **Pro** | 1M tweets/month | $5,000/month | Business apps |
| **Enterprise** | Custom | Custom | Data partners |

### 2FA Options

| Method | Security Level | Availability |
|---|---|---|
| **Security Key (FIDO2)** | ⭐⭐⭐⭐⭐ | All users |
| **Auth App (TOTP)** | ⭐⭐⭐⭐ | All users |
| **SMS** | ⭐⭐ | X Premium only (since 2023) |

---

## 3. API Security & Anti-Scraping

```mermaid
flowchart TB
    REQ3["📱 API Request"] --> AUTH3{"Authentication?"}
    AUTH3 -->|"No token"| REJECT2["❌ 401 Unauthorized"]
    AUTH3 -->|"Valid token"| SCOPE2{"Scopes OK?"}
    SCOPE2 -->|"Insufficient"| REJECT3["❌ 403 Forbidden"]
    SCOPE2 -->|"OK"| RL3{"Rate Limit?"}

    RL3 -->|"Within limit"| VALIDATE2["✅ Input Validation"]
    RL3 -->|"Exceeded"| RATE["⏳ 429 Too Many Requests<br/>+ Retry-After header"]

    VALIDATE2 --> BEHAVIOR{"Behavioral Check"}
    BEHAVIOR -->|"Normal"| PROCESS["✅ Process Request"]
    BEHAVIOR -->|"Suspicious<br/>(scraping pattern)"| FLAG2["🚩 Flag + Throttle"]

    style REJECT2 fill:#ff6b6b,color:#fff
    style REJECT3 fill:#ff6b6b,color:#fff
    style RATE fill:#ffd43b,color:#333
    style PROCESS fill:#51cf66,color:#fff
```

### Anti-Scraping Measures

| Measure | Mô tả |
|---|---|
| **API-only access** | Chỉ cho phép access qua official API |
| **Tiered rate limits** | Free tier rất hạn chế, ngăn mass scraping |
| **Behavioral detection** | ML phát hiện patterns scraping (sequential reads, no engagement) |
| **Token revocation** | Tự động revoke tokens vi phạm TOS |
| **IP reputation** | Block known scraping infrastructure |

### Bài Học Từ API Vulnerability (2021)

```mermaid
flowchart LR
    VULN["⚠️ API Bug 2021:<br/>Lookup phone/email → user_id"] --> EXPLOIT["🔴 Scrapers exploit<br/>enumerate emails/phones"]
    EXPLOIT --> LEAK["📊 200M+ records leaked"]
    LEAK --> FIX["🔧 Fix:<br/>1. Patch endpoint<br/>2. Rate limit by input<br/>3. Require auth for lookups"]

    style VULN fill:#ff6b6b,color:#fff
    style FIX fill:#51cf66,color:#fff
```

**Bài học:** Mọi endpoint trả về user data phải có rate limiting VÀ authentication, kể cả lookup endpoints.

---

## 4. Bot Detection & Spam Prevention

```mermaid
flowchart TB
    ACCOUNT["Account Activity"] --> SIGNALS["📊 Behavioral Signals"]

    SIGNALS --> SIG1["Posting frequency"]
    SIGNALS --> SIG2["Human interaction<br/>(screen tap, scroll)"]
    SIGNALS --> SIG3["Content patterns<br/>(repetitive, links)"]
    SIGNALS --> SIG4["Graph analysis<br/>(follow/follower ratio)"]
    SIGNALS --> SIG5["Device fingerprint"]
    SIGNALS --> SIG6["Account age vs activity"]

    SIG1 & SIG2 & SIG3 & SIG4 & SIG5 & SIG6 --> ML2["🤖 ML Classifier"]

    ML2 --> SCORE2{"Bot Score"}
    SCORE2 -->|"Low (human)"| NORMAL["✅ Normal"]
    SCORE2 -->|"Medium"| SHADOW2["👻 Reduce reach<br/>(shadow restrict)"]
    SCORE2 -->|"High (bot)"| SUSPEND["🚫 Suspend + CAPTCHA"]

    style NORMAL fill:#51cf66,color:#fff
    style SHADOW2 fill:#ffd43b,color:#333
    style SUSPEND fill:#ff6b6b,color:#fff
```

### Spam Prevention Pipeline

| Stage | Action | Examples |
|---|---|---|
| **Pre-publish** | Content filter | Spam links, known malware URLs |
| **Post-publish** | ML analysis | Repetitive tweets, engagement manipulation |
| **Reactive** | User reports + review | Mass report → human review |
| **Proactive** | Batch purge | Millions of bot accounts removed periodically |

---

## 5. Data Protection

```mermaid
graph TB
    subgraph "Data Classification"
        PUB["🌐 Public<br/>Tweets, profile, follower count"]
        INT["🔒 Internal<br/>Email (hashed), phone (hashed), IP logs"]
        CRIT["🔴 Critical<br/>Passwords (bcrypt), tokens, DMs"]
    end

    subgraph "Protection Measures"
        PUB --> P1["CDN cached, publicly accessible"]
        INT --> P2["Encrypted at rest, access controlled"]
        CRIT --> P3["HSM-stored keys, strict ACL, audit logged"]
    end

    subgraph "Encryption"
        E1["In transit: TLS 1.3 everywhere"]
        E2["At rest: AES-256 (Manhattan encrypted)"]
        E3["Secrets: Vault/HSM, auto-rotation"]
    end

    style PUB fill:#51cf66,color:#fff
    style INT fill:#ffd43b,color:#333
    style CRIT fill:#ff6b6b,color:#fff
```

---

## 6. Content Moderation

```mermaid
flowchart TB
    POST2["📝 Tweet Published"] --> HASH2["#️⃣ Hash Check<br/>(Known harmful media)"]
    HASH2 -->|Match| REMOVE["🚫 Auto-remove"]
    HASH2 -->|No match| AI2["🤖 AI Analysis"]

    AI2 --> CHECK3{"Content type?"}
    CHECK3 -->|"Hate speech"| LABEL["⚠️ Label + Reduce reach"]
    CHECK3 -->|"Misinformation"| NOTE["📝 Community Note"]
    CHECK3 -->|"Illegal"| REMOVE
    CHECK3 -->|"Safe"| PUBLISH2["✅ Published"]

    NOTE --> CROWD["👥 Community Notes<br/>(Crowd-sourced fact-check)"]

    style REMOVE fill:#ff6b6b,color:#fff
    style PUBLISH2 fill:#51cf66,color:#fff
```

**Community Notes** — Unique Twitter feature: crowd-sourced fact-checking thay vì chỉ dựa vào AI.

---

## 7. Security Monitoring

| Metric | SLI | Alert Threshold |
|---|---|---|
| Login failure rate | % failed logins | > 5% per minute |
| API error rate | 5xx responses | > 1% per endpoint |
| Scraping detection | Sequential reads without engagement | > 1000 reads/min/token |
| Bot creation rate | New accounts with bot signals | > baseline × 2 |
| DDoS traffic | Requests per second | > baseline × 10 |

---

## 8. So Sánh Security: Twitter vs Instagram

| Layer | Twitter/X | Instagram |
|---|---|---|
| **Auth protocol** | OAuth 1.0a + 2.0 (PKCE) | OAuth 2.0 |
| **2FA** | FIDO2, TOTP, SMS(paid) | TOTP, SMS, WhatsApp |
| **API access** | Paid tiers ($100-$5K/month) | Free (with review) |
| **Bot detection** | Human-tap behavioral | AI behavioral analysis |
| **Content moderation** | AI + Community Notes | AI + human reviewers |
| **DDoS** | 3rd-party + internal | Meta own infrastructure |
| **Unique risk** | API scraping (2021 breach) | Thundering herd on celebrity |
| **Open-source** | More transparent (formerly) | Less public |

---

## Mapping → NestJS

| Pattern | Twitter/X | NestJS Implementation |
|---|---|---|
| **OAuth 2.0 PKCE** | Native for API access | `@nestjs/passport` + `passport-oauth2` |
| **Tiered Rate Limit** | Free/Basic/Pro/Enterprise | `@nestjs/throttler` + Redis + user tier check |
| **Bot Detection** | Behavioral ML scoring | Custom middleware + `device-detector` |
| **Anti-Scraping** | Sequential read detection | Rate limit per-user + anomaly detection |
| **Content Hash** | Known harmful content DB | `crypto.createHash` + external DB (PhotoDNA) |
| **Community Notes** | Crowd-sourced moderation | Custom voting system + reputation score |
| **API Tiers** | Paid access levels | `@nestjs/passport` + Stripe subscription check |

> [!CAUTION]
> **Bài học #1 từ Twitter:** API endpoint nào trả về user data đều phải có rate limiting + authentication, kể cả lookup/search endpoints. Vulnerability 2021 là do thiếu rate limit trên lookup endpoint.
