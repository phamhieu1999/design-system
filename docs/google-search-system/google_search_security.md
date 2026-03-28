# Google Search - Security Analysis

> Google Search chống lại spam, SEO manipulation, malware distribution, và bảo vệ user privacy.

---

## Tổng Quan

```mermaid
graph TB
    subgraph "Layer 1: Web Spam"
        SPAM5["🗑️ Spam Detection<br/>(Content + link spam)"]
        MANUAL3["👤 Manual Actions<br/>(Search Quality team)"]
    end

    subgraph "Layer 2: Safe Browsing"
        SB2["🛡️ Safe Browsing<br/>(Malware, phishing, unwanted)"]
        WARN2["⚠️ Warning interstitials"]
    end

    subgraph "Layer 3: Content Quality"
        EAT["📊 E-E-A-T<br/>(Experience, Expertise,<br/>Authority, Trust)"]
        HELPFUL["📝 Helpful Content System"]
    end

    subgraph "Layer 4: User Privacy"
        ANON2["🔒 Anonymized search logs"]
        DEL["🗑️ Auto-delete (3-36 months)"]
        INCOG["🕶️ Incognito mode"]
    end

    subgraph "Layer 5: Infrastructure"
        DDOS2["🛡️ DDoS protection (Maglev)"]
        BORGGUARD["🔐 BorgGuard (auth)"]
        ENCRYPT8["🔐 Encryption everywhere"]
    end
```

---

## 1. Web Spam Detection

```mermaid
flowchart TB
    PAGE2["📄 Web page"] --> SIGNALS3["Spam Signals"]

    subgraph "Spam Types"
        KEYWORD["Keyword stuffing"]
        CLOAKING["Cloaking (show different<br/>content to Googlebot)"]
        LINK_FARM["Link farms / PBN"]
        DOORWAY["Doorway pages"]
        SCRAPE["Scraped content"]
        AUTO_GEN["AI-generated spam"]
    end

    SIGNALS3 --> ML13["🤖 SpamBrain (ML)"]
    ML13 --> DEMOTE["📉 Demote in rankings"]
    ML13 --> DEINDEX["🚫 De-index entirely"]
    ML13 --> MANUAL4["👤 Manual review queue"]

    NOTE18["SpamBrain: Neural network<br/>trained on known spam examples<br/>Catches new spam patterns automatically"]

    style ML13 fill:#EA4335,color:#fff
```

---

## 2. Google Safe Browsing

```mermaid
flowchart TB
    BROWSE["🌐 User browsing"] --> CHECK9["🛡️ Safe Browsing check<br/>(Client-side hash prefix)"]

    CHECK9 --> SAFE2{"In threat list?"}
    SAFE2 -->|"No"| PROCEED["✅ Load page"]
    SAFE2 -->|"Maybe (hash match)"| FULL_CHECK["Full hash check<br/>(Server-side)"]
    FULL_CHECK -->|"Safe"| PROCEED
    FULL_CHECK -->|"Threat"| INTERSTITIAL["⚠️ Warning page:<br/>'This site may harm your computer'"]

    subgraph "Threat Types"
        T1["🦠 Malware (auto-downloads)"]
        T2["🎣 Phishing (fake login)"]
        T3["📦 Unwanted software"]
        T4["🔒 Social engineering"]
    end

    NOTE19["Used by Chrome, Firefox, Safari<br/>Protects 4B+ devices worldwide"]

    style INTERSTITIAL fill:#EA4335,color:#fff
```

### Privacy-Preserving Lookup

```
1. Client hashes URL → SHA256
2. Sends first 4 bytes (prefix) to Google
3. Google returns all matching full hashes
4. Client checks locally → Google NEVER sees full URL
```

---

## 3. E-E-A-T & Content Quality

| Signal | Meaning | Example |
|---|---|---|
| **Experience** | First-hand experience | Product review from actual user |
| **Expertise** | Subject knowledge | Medical article by a doctor |
| **Authoritativeness** | Recognized authority | .gov or .edu domains |
| **Trustworthiness** | Reliable, honest | Secure site, accurate info |

---

## 4. So Sánh Security

| Layer | Google Search | YouTube | Stripe | Amazon |
|---|---|---|---|---|
| **Focus** | Web spam, malware | Copyright, harmful content | Payment fraud | Marketplace fraud |
| **Detection** | SpamBrain ML | Content ID | Radar ML | A-to-Z |
| **Unique** | Safe Browsing (4B devices) | CSAM detection | Cross-merchant intel | Chaotic storage |
| **Privacy** | Prefix hash lookup | Age verification | PCI tokenization | KMS encryption |

---

## Mapping → NestJS

| Pattern | Google | NestJS Implementation |
|---|---|---|
| **Safe Browsing** | Hash-prefix API | Google Safe Browsing API v4 |
| **Spam detection** | SpamBrain ML | Custom rules + ML microservice |
| **E-E-A-T scoring** | Quality rater + ML | Domain trust table + scoring |
| **Content quality** | Helpful Content System | Text analysis + readability score |
| **Rate limiting** | Maglev | `@nestjs/throttler` |
