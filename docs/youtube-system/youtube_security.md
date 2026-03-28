# YouTube - Security Analysis

> YouTube bảo vệ 800M+ videos, creator earnings, và 2.5B+ user accounts.

---

## Tổng Quan

```mermaid
graph TB
    subgraph "Layer 1: Network"
        DDOS3["🌍 Google Shield<br/>(DDoS at edge)"]
        TLS5["🔒 TLS 1.3 + QUIC"]
    end

    subgraph "Layer 2: Auth"
        GAUTH["🔑 Google Account (OAuth 2.0)"]
        MFA6["📱 Google 2FA / Passkeys"]
        SESSION3["🎫 Session management"]
    end

    subgraph "Layer 3: Content Protection"
        CID["🎵 Content ID<br/>(Copyright)"]
        HASH3["#️⃣ CSAM Hash matching"]
        AGE["🔞 Age-restricted content"]
    end

    subgraph "Layer 4: Anti-abuse"
        BOT2["🤖 Bot / view fraud detection"]
        SPAM3["🗑️ Comment spam"]
        SCAM["⚠️ Scam/phishing detection"]
    end

    subgraph "Layer 5: Creator Security"
        REVENUE["💰 Revenue protection"]
        CHANNEL["📺 Channel security"]
        API_KEY["🔑 API key management"]
    end
```

---

## 1. Content ID — Copyright Protection

```mermaid
flowchart TB
    UPLOAD5["📤 Video uploaded"] --> FINGERPRINT["🎵 Audio + Video Fingerprint<br/>(Extract features)"]
    FINGERPRINT --> MATCH{"Match against<br/>Content ID database<br/>(100M+ reference files)"}

    MATCH -->|"No match"| PUBLISH["✅ Publish normally"]
    MATCH -->|"Match found"| POLICY{"Rights holder policy?"}

    POLICY -->|"Monetize"| ADS2["💰 Ads on video<br/>(Revenue to rights holder)"]
    POLICY -->|"Block"| BLOCK2["🚫 Block in regions"]
    POLICY -->|"Track"| TRACK["📊 Track analytics"]

    subgraph "Content ID Scale"
        SCALE_1["100M+ reference files"]
        SCALE_2["800M+ videos scanned"]
        SCALE_3["$9B+ paid to rights holders"]
    end

    style ADS2 fill:#51cf66,color:#fff
    style BLOCK2 fill:#ff6b6b,color:#fff
```

**Content ID** scan mỗi video upload → so sánh với 100M+ audio/video fingerprints → tự động áp dụng policy.

---

## 2. View Fraud Detection

```mermaid
flowchart TB
    VIEW2["👁️ View event"] --> CHECKS["Multi-layer validation"]

    CHECKS --> C1["📱 Device fingerprint"]
    CHECKS --> C2["🌐 IP reputation"]
    CHECKS --> C3["⏱️ Watch duration (> 30s)"]
    CHECKS --> C4["🤖 Bot behavior patterns"]
    CHECKS --> C5["📊 Statistical anomaly"]
    CHECKS --> C6["🔄 Duplicate detection"]

    C1 & C2 & C3 & C4 & C5 & C6 --> SCORE3{"Fraud score"}
    SCORE3 -->|"Legitimate"| COUNT2["✅ Count view"]
    SCORE3 -->|"Suspicious"| FREEZE["❄️ Freeze count<br/>(Investigate)"]
    SCORE3 -->|"Fraudulent"| DISCARD["🚫 Discard<br/>+ Flag account"]

    style DISCARD fill:#ff6b6b,color:#fff
```

**Why critical:** View counts directly determine **creator revenue** (ad revenue sharing). Fake views = fake money.

---

## 3. Content Moderation Pipeline

```mermaid
flowchart TB
    CONTENT4["📤 New upload"] --> AI_MOD["🤖 AI Moderation"]

    AI_MOD --> CHECK4{"AI Classification"}
    CHECK4 -->|"Violence/Hate"| RESTRICT["⚠️ Age-restrict"]
    CHECK4 -->|"CSAM"| REMOVE2["🚫 Remove + Report to NCMEC"]
    CHECK4 -->|"Spam/Scam"| BLOCK3["🚫 Block"]
    CHECK4 -->|"Borderline"| HUMAN["👤 Human Review queue"]
    CHECK4 -->|"Safe"| PUBLISH2["✅ Publish"]

    HUMAN --> DECISION{"Decision"}
    DECISION -->|"Violation"| STRIKE["⚠️ Channel Strike<br/>(3 strikes = termination)"]
    DECISION -->|"OK"| PUBLISH2

    subgraph "Scale"
        N1["10M+ videos removed/quarter"]
        N2[">94% found by AI before human report"]
    end

    style REMOVE2 fill:#ff6b6b,color:#fff
```

---

## 4. API Security

| Measure | Implementation |
|---|---|
| **API Keys** | Per-project keys, quotas per key |
| **OAuth 2.0** | Standard Google OAuth for user data |
| **Rate Limiting** | Quota units per day (10,000 default) |
| **Abuse detection** | Anomalous API usage → throttle/ban |
| **Scopes** | Granular (read, upload, manage, etc.) |

---

## 5. So Sánh Security: YouTube vs Others

| Layer | YouTube | Netflix | Instagram | Twitter |
|---|---|---|---|---|
| **Focus** | Copyright + creator revenue | Content piracy (DRM) | Account security | API/bot abuse |
| **Content protection** | Content ID (fingerprint) | DRM + watermark | AI moderation | Community Notes |
| **View integrity** | Multi-layer fraud detection | N/A (subscription) | N/A | N/A |
| **Unique** | $9B+ paid to rights holders | Forensic watermark | ML login risk | FIDO2 key |
| **Auth** | Google Account | Custom OAuth | Meta OAuth | X OAuth |
| **Cloud** | Google Cloud | AWS | Meta DCs | Google Cloud |

---

## Mapping → NestJS

| Pattern | YouTube | NestJS Implementation |
|---|---|---|
| **Content ID** | Audio/video fingerprint | `chromaprint` + custom DB matching |
| **View fraud** | Multi-signal validation | Redis rate counter + ML scoring service |
| **Content moderation** | AI pipeline | Google Vision API / AWS Rekognition |
| **API quotas** | Per-key daily limits | `@nestjs/throttler` + Redis per-API-key |
| **Strike system** | 3-strike channel policy | Database counter + automated enforcement |
