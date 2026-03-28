# Netflix - Security Analysis

> Netflix bảo vệ 260M+ accounts, DRM content trị giá hàng tỷ USD, và streaming data toàn cầu.

---

## Tổng Quan: Defense in Depth

```mermaid
graph TB
    subgraph "Layer 1: Edge & Network"
        DDOS2["🌍 DDoS Protection (AWS Shield)"]
        WAF3["🛡️ WAF (AWS WAF + custom)"]
        TLS4["🔒 TLS 1.3 everywhere"]
    end

    subgraph "Layer 2: Authentication"
        AUTH2["🔑 Netflix Auth (custom OAuth)"]
        MFA4["📱 MFA + Device Management"]
        TOKEN["🎫 Short-lived tokens + refresh"]
    end

    subgraph "Layer 3: Authorization"
        RBAC["👑 RBAC (profiles/parental)"]
        DRM["🔐 DRM (Widevine/FairPlay/PlayReady)"]
        GEO["🌍 Geo-restrictions"]
    end

    subgraph "Layer 4: Application"
        ZUUL_SEC["🚪 Zuul Security Filters"]
        INPUT2["📝 Input Validation"]
        RATE2["⚖️ Rate Limiting"]
    end

    subgraph "Layer 5: Data"
        ENC2["🔐 Encryption at Rest (S3 SSE)"]
        SECRETS["🗝️ Secret Management"]
        LOGGING2["📋 Audit Logging"]
    end

    subgraph "Layer 6: Content Protection"
        DRM2["🎬 Multi-DRM"]
        WATERMARK["💧 Forensic Watermarking"]
        TAMPER["🛡️ Client Tamper Detection"]
    end
```

---

## 1. DRM — Digital Rights Management

```mermaid
sequenceDiagram
    participant Client as 📱 Netflix Client
    participant Server as Netflix Server
    participant License as License Server
    participant CDM as CDM (Device)

    Client->>Server: Play request (title_id)
    Server->>Server: Check entitlement (subscription active?)
    Server->>Server: Check geo-restriction
    Server-->>Client: Manifest + License URL

    Client->>CDM: Initialize DRM module
    CDM->>License: Request license (device certificate)
    License->>License: Validate device + rights
    License-->>CDM: Decryption key (time-limited)
    
    Client->>CDM: Decrypt video segments
    CDM-->>Client: Decoded frames → render

    Note over Client,CDM: Keys NEVER leave CDM hardware<br/>Video decoded in secure pipeline
```

### Multi-DRM Strategy

| DRM | Platform | Security Level |
|---|---|---|
| **Widevine L1** | Android, Chrome, Smart TVs | Hardware-backed (4K allowed) |
| **Widevine L3** | Desktop Chrome | Software (720p max) |
| **FairPlay** | iOS, macOS, Apple TV | Hardware-backed |
| **PlayReady** | Windows, Xbox | Hardware-backed |

### Forensic Watermarking

```mermaid
flowchart LR
    VIDEO2["🎬 Video Stream"] --> WATERMARK2["💧 Invisible Watermark<br/>(Unique per account)"]
    WATERMARK2 --> USER4["👤 User watches"]
    
    USER4 --> LEAK{"Content leaked?"}
    LEAK -->|Yes| EXTRACT["🔍 Extract watermark"]
    EXTRACT --> TRACE["🎯 Trace to exact account"]
    TRACE --> ACTION2["⚖️ Legal action + terminate"]

    style TRACE fill:#ff6b6b,color:#fff
```

---

## 2. Authentication & Session

```mermaid
flowchart TB
    LOGIN["🔑 Login<br/>(email + password)"] --> VERIFY2["Verify credentials"]
    VERIFY2 --> MFA5{"MFA required?<br/>(New device/location)"}
    MFA5 -->|Yes| OTP2["📱 Email/SMS verification"]
    MFA5 -->|No| TOKEN2["Generate tokens"]
    OTP2 --> TOKEN2

    TOKEN2 --> ACCESS["Access Token<br/>(short-lived, 15min)"]
    TOKEN2 --> REFRESH2["Refresh Token<br/>(long-lived, device-bound)"]

    subgraph "Device Management"
        DEV["Max devices per plan:<br/>• Basic: 1 stream<br/>• Standard: 2 streams<br/>• Premium: 4 streams"]
        KICK["Can remove devices remotely"]
    end

    subgraph "Password Sharing Detection"
        DETECT2["📍 IP location analysis"]
        DETECT2 --> HOUSEHOLD["🏠 Household verification"]
        HOUSEHOLD --> EXTRA["💳 Extra member fee"]
    end
```

---

## 3. API Security (Zuul Filters)

```mermaid
flowchart TB
    REQ5["📱 API Request"] --> ZUUL3["🚪 Zuul Gateway"]

    subgraph "Zuul Security Filter Chain"
        F1["1. TLS Termination"]
        F2["2. Authentication Filter"]
        F3["3. Rate Limit Filter"]
        F4["4. Device Fingerprint"]
        F5["5. Geo-check Filter"]
        F6["6. Request Validation"]
        F7["7. CORS Filter"]
        F1 --> F2 --> F3 --> F4 --> F5 --> F6 --> F7
    end

    ZUUL3 --> F1
    F7 --> BACKEND["Backend Services"]

    style ZUUL3 fill:#4c6ef5,color:#fff
```

---

## 4. Data Protection

| Data Type | Protection | Storage |
|---|---|---|
| **Passwords** | bcrypt (cost factor 12) | Aurora (encrypted) |
| **Payment info** | PCI DSS, tokenized | 3rd-party processor |
| **Viewing history** | Encrypted at rest | Cassandra (encrypted) |
| **Content files** | AES-128/256 (DRM) | S3 SSE + OCA |
| **API secrets** | Netflix Vault (custom) | Auto-rotation |
| **Logs** | PII scrubbed | S3 + Elasticsearch |

---

## 5. Content Security Pipeline

```mermaid
flowchart TB
    CONTENT3["🎬 New Content<br/>(Pre-release)"] --> ENCRYPT4["🔐 Encrypt during encoding"]
    ENCRYPT4 --> SEGMENT2["✂️ Segment + package"]
    SEGMENT2 --> WATERMARK3["💧 Watermark variants"]
    WATERMARK3 --> DISTRIBUTE["📤 Distribute to OCAs<br/>(Encrypted at rest)"]

    subgraph "Playback Security"
        HW["Hardware DRM<br/>(CDM in TEE)"]
        HDCP["HDCP 2.2<br/>(Output protection)"]
        NO_RECORD["🚫 Screen capture blocked<br/>(On supported devices)"]
    end

    style ENCRYPT4 fill:#4c6ef5,color:#fff
```

---

## 6. So Sánh Security: Netflix vs Others

| Layer | Netflix | Instagram | Twitter | WhatsApp |
|---|---|---|---|---|
| **Focus** | Content protection (DRM) | Account security | API security | Message encryption |
| **Auth** | Custom OAuth + device mgmt | OAuth 2.0 | OAuth 2.0 PKCE | Phone + OTP |
| **Encryption** | DRM (Widevine/FairPlay) | TLS | TLS | Signal E2EE |
| **Unique** | Forensic watermarking | ML login risk | Community Notes | Forward secrecy |
| **Threat model** | Content piracy | Account takeover | Bot/scraping | Surveillance |
| **Cloud** | AWS (full) | Meta DCs | Google Cloud | Meta DCs |

---

## Mapping → NestJS

| Pattern | Netflix | NestJS Implementation |
|---|---|---|
| **Gateway security** | Zuul filter chain | NestJS Guards + Pipes + Interceptors |
| **Token auth** | Custom OAuth + refresh | `@nestjs/jwt` + refresh token rotation |
| **Device management** | Device registry + limits | Redis `SET` per user + limit middleware |
| **Rate limiting** | Per-device/per-user | `@nestjs/throttler` with custom storage |
| **DRM** | Widevine/FairPlay | HLS.js + Shaka Player (client-side) |
| **Watermarking** | Per-account invisible | Server-side video overlay pipeline |
| **Secret management** | Netflix Vault | AWS Secrets Manager / HashiCorp Vault |
