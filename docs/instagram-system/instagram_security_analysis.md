# Phân Tích Bảo Mật Hệ Thống Instagram

> Phân tích 8 lớp bảo mật của Instagram — từ network edge đến application logic, bảo vệ 2+ tỷ users trước hàng triệu cuộc tấn công mỗi ngày.

---

## Tổng Quan: Defense in Depth

```mermaid
graph TB
    subgraph "Layer 1: Network Edge"
        CDN["🌍 CDN + Anycast<br/>DDoS Absorption"]
        WAF["🛡️ WAF<br/>Request Filtering"]
        TLS["🔒 TLS 1.3<br/>Certificate Pinning"]
    end

    subgraph "Layer 2: Authentication"
        OAUTH["🔑 OAuth 2.0"]
        MFA["📱 2FA / MFA"]
        SESSION["🎫 Session Management"]
    end

    subgraph "Layer 3: API Gateway"
        RL["⚖️ Rate Limiting"]
        VALIDATE["✅ Input Validation"]
        AUTHZ["🔐 Authorization (RBAC)"]
    end

    subgraph "Layer 4: Application"
        CSRF["🛡️ CSRF / XSS Protection"]
        INJECT["💉 Injection Prevention"]
        ENCRYPT["🔐 Data Encryption"]
    end

    subgraph "Layer 5: Data"
        ATREST["💾 Encryption at Rest"]
        ACCESS["🔒 Access Control"]
        AUDIT["📋 Audit Logging"]
    end

    subgraph "Layer 6: Monitoring"
        DETECT["👁️ Anomaly Detection"]
        SIEM["📊 SIEM"]
        RESPOND["🚨 Incident Response"]
    end

    CDN --> WAF --> TLS --> OAUTH --> MFA --> SESSION --> RL --> VALIDATE --> AUTHZ --> CSRF --> INJECT --> ENCRYPT --> ATREST --> ACCESS --> AUDIT --> DETECT --> SIEM --> RESPOND

    style CDN fill:#4c6ef5,color:#fff
    style OAUTH fill:#7950f2,color:#fff
    style RL fill:#ae3ec9,color:#fff
    style CSRF fill:#e64980,color:#fff
    style ATREST fill:#ff922b,color:#fff
    style DETECT fill:#ff6b6b,color:#fff
```

---

## 1. Network Security Layer

### 1.1 TLS & Certificate Pinning

```mermaid
sequenceDiagram
    participant App as 📱 Instagram App
    participant Edge as 🌍 Meta Edge Server
    participant Origin as 🏢 Origin Server

    App->>App: Check pinned certificate hash
    App->>Edge: TLS 1.3 Handshake
    Edge-->>App: Server Certificate
    App->>App: Validate cert against pinned hashes
    
    alt Certificate VALID & PINNED
        App->>Edge: ✅ Encrypted request
        Edge->>Origin: Internal mTLS
    else Certificate INVALID
        App->>App: ❌ Block connection
        App->>App: Report to security team
    end
```

| Kỹ thuật | Mô tả |
|---|---|
| **TLS 1.3** | Encryption in-transit cho mọi kết nối |
| **Certificate Pinning** | App chỉ chấp nhận certs cụ thể → chống MITM |
| **mTLS (internal)** | Service-to-service authentication bằng mutual TLS |
| **HSTS** | Force HTTPS, chống downgrade attacks |

### 1.2 DDoS Protection

```mermaid
flowchart LR
    ATTACK["🔴 DDoS Attack<br/>100M+ req/s"] --> ANYCAST["🌍 Anycast Network<br/>Phân tán traffic<br/>tới nhiều DC"]
    
    ANYCAST --> EDGE1["Edge DC 1<br/>Filter 40%"]
    ANYCAST --> EDGE2["Edge DC 2<br/>Filter 30%"]
    ANYCAST --> EDGE3["Edge DC 3<br/>Filter 30%"]
    
    EDGE1 & EDGE2 & EDGE3 --> ML_FILTER["🤖 ML-based Filtering<br/>• Pattern detection<br/>• Bot fingerprinting<br/>• Reputation scoring"]
    
    ML_FILTER -->|"1% legitimate"| ORIGIN2["✅ Origin Servers"]
    ML_FILTER -->|"99% malicious"| DROP["❌ Drop"]

    style ATTACK fill:#ff6b6b,color:#fff
    style DROP fill:#ff6b6b,color:#fff
    style ORIGIN2 fill:#51cf66,color:#fff
```

---

## 2. Authentication & Identity

### 2.1 OAuth 2.0 Flow (Third-party API)

```mermaid
sequenceDiagram
    actor User as 👤 User
    participant App3rd as 🏢 3rd Party App
    participant IG as 🔑 Instagram Auth Server
    participant API as 📡 Instagram API

    User->>App3rd: Click "Login with Instagram"
    App3rd->>IG: Redirect to /oauth/authorize?scope=...
    IG->>User: Login form + consent screen
    User->>IG: Credentials + approve scopes
    IG->>App3rd: Redirect with authorization_code
    App3rd->>IG: POST /oauth/token (code + client_secret)
    IG-->>App3rd: access_token (app-scoped, short-lived)
    App3rd->>API: GET /me?access_token=xxx
    API-->>App3rd: User data (scoped)
    
    Note over App3rd,API: Token là app-scoped:<br/>1 user + 1 app = 1 unique token
```

### 2.2 Multi-Factor Authentication & Login Detection

```mermaid
flowchart TB
    LOGIN["🔑 Login Attempt"] --> CRED{"Credentials correct?"}
    CRED -->|No| BLOCK1["❌ Invalid credentials<br/>Rate limit tracking"]
    CRED -->|Yes| RISK["🤖 Risk Assessment"]
    
    RISK --> R_CHECK{"Risk Score?"}
    
    R_CHECK -->|"LOW<br/>(known device, location)"| ALLOW["✅ Login allowed"]
    R_CHECK -->|"MEDIUM<br/>(new device OR location)"| MFA["📱 Require 2FA"]
    R_CHECK -->|"HIGH<br/>(new device + location + VPN)"| CHALLENGE["🔒 Security Challenge<br/>Video selfie / email verify"]

    MFA --> MFA_TYPE{"2FA Method"}
    MFA_TYPE --> TOTP["📲 Auth App (TOTP)"]
    MFA_TYPE --> SMS["📨 SMS Code"]
    MFA_TYPE --> WA["💬 WhatsApp"]

    TOTP & SMS & WA --> VERIFY{"Code correct?"}
    VERIFY -->|Yes| ALLOW
    VERIFY -->|No| BLOCK2["❌ Block + Alert user"]

    style BLOCK1 fill:#ff6b6b,color:#fff
    style BLOCK2 fill:#ff6b6b,color:#fff
    style ALLOW fill:#51cf66,color:#fff
    style CHALLENGE fill:#ffd43b,color:#333
```

**Risk Assessment Signals:**

| Signal | Low Risk | Medium Risk | High Risk |
|---|---|---|---|
| Device | Known | New | New |
| Location | Same city | Same country | Different country |
| IP | Residential | Datacenter | Known VPN/Tor |
| Behavior | Normal | Slightly unusual | Automated/bot |
| Time | Usual hours | Off-hours | Never-before |

---

## 3. API Security & Rate Limiting

```mermaid
flowchart TB
    REQ["📱 API Request"] --> L1

    subgraph "Layer 1: Edge Rate Limit"
        L1["IP-based<br/>Token Bucket<br/>1000 req/min per IP"]
    end
    
    L1 --> L2
    
    subgraph "Layer 2: User Rate Limit"
        L2["User-based (JWT)<br/>Sliding Window<br/>200 req/min per user"]
    end
    
    L2 --> L3
    
    subgraph "Layer 3: Endpoint Rate Limit"
        L3["Score-based per endpoint"]
        L3A["GET /feed → 1 point"]
        L3B["POST /media → 5 points"]
        L3C["POST /like → 2 points"]
        L3 --- L3A & L3B & L3C
        L3D["Budget: 200 points/hour"]
    end
    
    L3 --> L4
    
    subgraph "Layer 4: Adaptive Throttling"
        L4{"System Load?"}
        L4 -->|"Normal"| PASS["✅ Process"]
        L4 -->|"High"| SLOW["🐢 Slow down (add latency)"]
        L4 -->|"Critical"| SHED["❌ Load shedding<br/>Drop low-priority reqs"]
    end

    style PASS fill:#51cf66,color:#fff
    style SLOW fill:#ffd43b,color:#333
    style SHED fill:#ff6b6b,color:#fff
```

### API Security Checklist

| Check | Implementation |
|---|---|
| **Authentication** | OAuth 2.0 Bearer tokens, app-scoped |
| **Input Validation** | Schema validation mọi request body |
| **SQL Injection** | Parameterized queries (Django ORM) |
| **XSS** | Output encoding, CSP headers |
| **CSRF** | Token-based, SameSite cookies |
| **CORS** | Whitelist specific origins |
| **IDOR** | Ownership check trên mọi resource access |

---

## 4. Data Protection

### 4.1 Encryption Strategy

```mermaid
graph TB
    subgraph "In Transit"
        T1["Client ↔ Edge: TLS 1.3"]
        T2["Edge ↔ Service: mTLS"]
        T3["Service ↔ Service: mTLS"]
        T4["Service ↔ DB: TLS + Auth"]
    end

    subgraph "At Rest"
        R1["Database: AES-256 (TDE)"]
        R2["Media Files: Encrypted blob storage"]
        R3["Backups: Encrypted + access controlled"]
        R4["Logs: PII redacted / encrypted"]
    end

    subgraph "In Use"
        U1["Memory: Secure enclaves (SGX)"]
        U2["Secrets: Vault / HSM"]
        U3["Keys: Automatic rotation"]
    end
```

### 4.2 Data Classification & Access Control

```mermaid
flowchart LR
    subgraph "PUBLIC"
        A["Username<br/>Profile Photo<br/>Public Posts"]
    end
    
    subgraph "INTERNAL"
        B["Email (hashed)<br/>Phone (hashed)<br/>Activity Logs"]
    end
    
    subgraph "RESTRICTED"
        C["Passwords (bcrypt)<br/>Payment Data<br/>Auth Tokens"]
    end
    
    subgraph "Access Control"
        A -->|"Anyone"| AC1["Open API"]
        B -->|"Authenticated + scoped"| AC2["Internal Services Only"]
        C -->|"Need-to-know + MFA"| AC3["Security Team + HSM"]
    end

    style A fill:#51cf66,color:#fff
    style B fill:#ffd43b,color:#333
    style C fill:#ff6b6b,color:#fff
```

---

## 5. Application Security

### 5.1 Input Validation Pipeline

```mermaid
flowchart LR
    INPUT["User Input"] --> S1["1. Schema Validation<br/>(type, length, format)"]
    S1 --> S2["2. Sanitization<br/>(strip HTML/scripts)"]
    S2 --> S3["3. Business Rules<br/>(caption ≤ 2200 chars)"]
    S3 --> S4["4. Content Policy<br/>(AI moderation)"]
    S4 --> STORE["💾 Safe to Store"]

    style INPUT fill:#ff6b6b,color:#fff
    style STORE fill:#51cf66,color:#fff
```

### 5.2 Security Headers

| Header | Value | Mục đích |
|---|---|---|
| `Content-Security-Policy` | `script-src 'self'` | Chống XSS, chỉ cho phép scripts từ domain |
| `X-Frame-Options` | `DENY` | Chống Clickjacking |
| `X-Content-Type-Options` | `nosniff` | Chống MIME-type sniffing |
| `Strict-Transport-Security` | `max-age=31536000` | Force HTTPS 1 năm |
| `Referrer-Policy` | `strict-origin` | Giới hạn referrer info |

---

## 6. Content Moderation & AI Security

```mermaid
flowchart TB
    UPLOAD["📸 Content Upload"] --> HASH["#️⃣ Image Hash Check<br/>(Known CSAM database)"]
    
    HASH -->|"Match"| BLOCK_REPORT["🚫 Block + Report to NCMEC"]
    HASH -->|"No match"| AI["🤖 AI Content Analysis"]
    
    AI --> AI_CHECK{"Content Type?"}
    AI_CHECK -->|"Violence/Nudity"| FLAG["🚩 Flag for review"]
    AI_CHECK -->|"Spam/Scam"| SHADOW["👻 Shadow ban"]
    AI_CHECK -->|"Safe"| PUBLISH["✅ Publish"]
    
    subgraph "Behavioral Analysis"
        BA1["📊 Account age vs activity"]
        BA2["🔗 Connection patterns"]
        BA3["📍 Geo anomalies"]
        BA4["⏰ Time-based patterns"]
    end
    
    BA1 & BA2 & BA3 & BA4 --> RISK2{"Risk Score"}
    RISK2 -->|"High"| RESTRICT["⚠️ Restrict account"]
    RISK2 -->|"Normal"| CONTINUE["✅ Normal operation"]

    style BLOCK_REPORT fill:#ff6b6b,color:#fff
    style PUBLISH fill:#51cf66,color:#fff
```

---

## 7. Security Monitoring & Incident Response

```mermaid
flowchart TB
    subgraph "Data Sources"
        S1["API Logs"]
        S2["Auth Events"]
        S3["Network Traffic"]
        S4["DB Access Logs"]
    end

    S1 & S2 & S3 & S4 --> SIEM2["📊 SIEM Platform<br/>(Real-time aggregation)"]
    
    SIEM2 --> DETECT2["🤖 Anomaly Detection ML"]
    
    DETECT2 --> ALERT{"Alert Level?"}
    
    ALERT -->|"P1 Critical<br/>(data breach)"| IR1["🚨 Incident Commander<br/>< 15 min response"]
    ALERT -->|"P2 High<br/>(mass attack)"| IR2["⚠️ On-call Engineer<br/>< 1 hour response"]
    ALERT -->|"P3 Medium<br/>(suspicious pattern)"| IR3["📋 Security Queue<br/>< 24 hour response"]

    IR1 --> PLAY["Playbook Execution:<br/>1. Isolate<br/>2. Assess<br/>3. Contain<br/>4. Remediate<br/>5. Post-mortem"]

    style IR1 fill:#ff6b6b,color:#fff
    style IR2 fill:#ffd43b,color:#333
    style IR3 fill:#4c6ef5,color:#fff
```

---

## 8. Áp Dụng Cho NestJS Microservices

| Layer | Instagram | NestJS Implementation |
|---|---|---|
| **TLS** | TLS 1.3 + cert pinning | Nginx/Traefik TLS termination |
| **Auth** | OAuth 2.0 + 2FA | `@nestjs/passport` + `passport-jwt` + `speakeasy` (TOTP) |
| **Rate Limit** | Multi-layer token bucket | `@nestjs/throttler` + Redis store |
| **Input Validation** | Schema validation | `class-validator` + `class-transformer` |
| **CSRF** | Token-based | `csurf` middleware |
| **Helmet** | Security headers | `@nestjs/helmet` (CSP, HSTS, X-Frame-Options) |
| **Encryption** | AES-256 at rest | `crypto` module + `argon2` for passwords |
| **IDOR** | Ownership guards | Custom `@OwnerGuard()` decorator |
| **Logging** | SIEM + anomaly detect | `winston` + ELK Stack + `@nestjs/terminus` health |
| **Circuit Breaker** | Internal protection | `opossum` library |

> [!IMPORTANT]
> **Nguyên tắc bảo mật #1**: *"Security is not a feature, it's a property"* — Bảo mật phải được thiết kế vào kiến trúc, không phải thêm vào sau.
