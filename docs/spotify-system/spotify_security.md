# Spotify - Security Analysis

> Spotify bảo vệ 600M+ accounts, DRM cho 100M+ tracks, licensing agreements toàn cầu.

---

## Tổng Quan

```mermaid
graph TB
    subgraph "Layer 1: Network"
        TLS8["🔒 TLS 1.3"]
        CDN_SEC["🌍 CDN Edge Security"]
    end

    subgraph "Layer 2: Auth"
        OAUTH3["🔑 OAuth 2.0 + PKCE"]
        SOCIAL["📱 Social login (Google, Facebook, Apple)"]
        MFA9["📱 MFA (optional)"]
    end

    subgraph "Layer 3: Content Protection"
        DRM3["🔐 Widevine DRM<br/>(Offline content)"]
        LICENSE2["📜 Licensing engine<br/>(Per-country rights)"]
        ROYALTY["💰 Royalty calculation"]
    end

    subgraph "Layer 4: Account Security"
        CRED["🔑 Credential stuffing protection"]
        BOT3["🤖 Bot / fake stream detection"]
        FAMILY["👨‍👩‍👧‍👦 Family plan address verification"]
    end

    subgraph "Layer 5: Data"
        GDPR2["🇪🇺 GDPR compliance"]
        DATA_PORT["📦 Data portability"]
        LISTEN_HISTORY["🔒 Listening history privacy"]
    end
```

---

## 1. Content DRM & Licensing

```mermaid
flowchart TB
    UPLOAD6["🎵 Label uploads track"] --> INGEST3["📥 Ingest Service"]
    INGEST3 --> ENCODE3["🔀 Encode to Ogg Vorbis<br/>(Multiple bitrates)"]
    ENCODE3 --> ENCRYPT7["🔐 Encrypt (Widevine DRM)"]
    ENCRYPT7 --> STORE2["📦 GCS (Encrypted at rest)"]

    PLAY2["▶️ User plays track"] --> LICENSE3["📜 License check"]
    LICENSE3 --> GEO_CHECK["🌍 Geo-check:<br/>Track available in user's country?"]
    GEO_CHECK -->|"Yes"| DECRYPT3["🔓 Decrypt key → play"]
    GEO_CHECK -->|"No"| UNAVAIL["❌ 'Not available in your region'"]

    subgraph "Licensing Complexity"
        L1["Same track may have different:<br/>• Rights holders per country<br/>• Royalty rates<br/>• Availability windows"]
    end
```

---

## 2. Fake Stream Detection

```mermaid
flowchart TB
    STREAM4["🎵 Play event"] --> ANALYSIS2["🤖 Stream Quality Analysis"]

    ANALYSIS2 --> CHECK7{"Legitimate?"}
    CHECK7 -->|"Real listener"| COUNT3["✅ Count toward royalties"]
    CHECK7 -->|"Bot / fake"| DISCARD2["🚫 Don't count<br/>+ Flag account"]

    subgraph "Detection Signals"
        DS1["⏱️ Play duration (< 30s = no count)"]
        DS2["🔄 Loop patterns (same track 100x)"]
        DS3["📱 Device fingerprint"]
        DS4["🌐 IP anomaly (datacenter IPs)"]
        DS5["👤 Account age + history"]
        DS6["🎵 Playlist composition (all 1 artist)"]
    end

    style DISCARD2 fill:#ff6b6b,color:#fff
```

**Why critical:** Fake streams steal royalties from real artists. Spotify removes fake streams retroactively → protects artist earnings.

---

## 3. Account Security

| Threat | Protection |
|---|---|
| **Credential stuffing** | Rate limiting + CAPTCHA + leaked password check |
| **Account sharing** | Device limit (6 devices offline, 1 stream per account) |
| **Family plan abuse** | Address verification (GPS + billing address) |
| **Premium fraud** | Payment verification + trial restrictions |
| **Session hijacking** | Short-lived tokens + refresh rotation |

---

## 4. So Sánh Security: Spotify vs Others

| Layer | Spotify | Netflix | YouTube | Stripe |
|---|---|---|---|---|
| **Focus** | Stream integrity + licensing | Content piracy | Copyright | Payment fraud |
| **DRM** | Widevine (offline) | Widevine + FairPlay | N/A (free) | N/A |
| **Content protection** | Fake stream detection | Forensic watermark | Content ID | Radar ML |
| **Unique** | Per-country licensing | Multi-DRM | $9B rights payments | Cross-merchant fraud |
| **Revenue model** | Ads + Premium | Subscription | Ads + Premium | Transaction fee |

---

## Mapping → NestJS

| Pattern | Spotify | NestJS Implementation |
|---|---|---|
| **Fake stream detection** | ML + rules | Kafka consumer + anomaly rules |
| **Licensing/geo-check** | Per-country rights DB | PostgreSQL + `geoip-lite` |
| **DRM** | Widevine | CDN-level DRM (CloudFront) |
| **Account limits** | Device tracking | Redis SET per user (max 6) |
| **GDPR** | Data export/delete | Bulk export endpoint + soft delete |
