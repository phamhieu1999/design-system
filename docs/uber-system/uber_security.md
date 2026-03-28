# Uber - Security Analysis

> Uber bảo vệ 130M+ accounts, xử lý $40B+ giao dịch/năm, và dữ liệu vị trí nhạy cảm.

---

## Tổng Quan

```mermaid
graph TB
    subgraph "Layer 1: Network"
        DDOS4["🌍 DDoS Protection"]
        TLS6["🔒 TLS 1.3 + Certificate Pinning"]
    end

    subgraph "Layer 2: Auth"
        AUTH4["🔑 OAuth 2.0 + Phone verification"]
        MFA7["📱 MFA (SMS, Auth app)"]
        DEVICE2["📲 Trusted device management"]
    end

    subgraph "Layer 3: Payment"
        PCI["💳 PCI DSS Level 1"]
        TOKENIZE["🔐 Payment tokenization"]
        PENNY["💰 Penny-drop verification"]
    end

    subgraph "Layer 4: Fraud"
        MASTERMIND["🧠 Mastermind Rules Engine"]
        ML_FRAUD["🤖 ML Fraud Scoring"]
        RADAR["📡 RADAR Anomaly Detection"]
    end

    subgraph "Layer 5: Location Privacy"
        TRIP_DATA["📍 Trip data anonymization"]
        RETENTION["⏰ Data retention policies"]
        ACCESS["🔑 Strict access controls"]
    end

    subgraph "Layer 6: Physical Safety"
        SOS["🆘 Emergency SOS"]
        SHARE_LOC["📍 Share trip with contacts"]
        VERIFY2["✅ Driver verification"]
    end
```

---

## 1. Payment Security

```mermaid
flowchart TB
    PAY["💳 Add payment method"] --> TOKEN3["🔐 Tokenize via PSP<br/>(Stripe/Braintree)"]
    TOKEN3 --> STORE_TOKEN["Store token<br/>(NOT raw card number!)"]

    RIDE_END["🏁 Ride ends"] --> CHARGE["Calculate fare"]
    CHARGE --> AUTH5["🔑 Authorize via token"]
    AUTH5 --> PSP["💳 Payment processor"]
    PSP -->|"Approved"| CONFIRM["✅ Charge confirmed"]
    PSP -->|"Declined"| RETRY2["🔄 Retry / alt method"]

    subgraph "Penny-Drop Verification"
        NEW_CARD["New card added"] --> HOLD["Authorization hold: $X.XX<br/>(Random small amount)"]
        HOLD --> USER_VERIFY["User enters exact amount<br/>on bank statement"]
        USER_VERIFY --> VERIFIED["✅ Card ownership verified"]
    end

    style TOKEN3 fill:#4c6ef5,color:#fff
```

---

## 2. Fraud Detection — Mastermind + RADAR

```mermaid
flowchart TB
    EVENT3["🔔 Event<br/>(Login, ride, payment)"] --> PARALLEL2

    subgraph "Parallel Evaluation"
        RULES["📋 Mastermind Rules<br/>(Human-authored)"]
        ML6["🤖 ML Risk Score<br/>(Behavioral model)"]
    end

    PARALLEL2 --> RULES & ML6
    RULES & ML6 --> COMBINE["Combine signals"]

    COMBINE --> RISK{"Risk Level"}
    RISK -->|"Low"| ALLOW3["✅ Allow"]
    RISK -->|"Medium"| CHALLENGE["🔐 Challenge<br/>(OTP, selfie, CAPTCHA)"]
    RISK -->|"High"| BLOCK4["🚫 Block + review"]

    subgraph "RADAR: Anomaly Detection"
        TIMESERIES["📈 Time-series monitoring<br/>(Fraud volume per category)"]
        TIMESERIES --> ANOMALY["🚨 Anomaly detected?"]
        ANOMALY --> AUTO_RULE["Auto-generate rule"]
        AUTO_RULE --> ANALYST["👤 Analyst verifies"]
        ANALYST --> DEPLOY2["🚀 Deploy rule"]
    end

    style BLOCK4 fill:#ff6b6b,color:#fff
```

### Common Fraud Patterns

| Pattern | Detection | Action |
|---|---|---|
| **Stolen credit card** | TC40/SAFE early signals | Block card + refund |
| **Promo abuse** | Device fingerprint + account linking | Ban linked accounts |
| **GPS spoofing** | Accelerometer + network signals | Flag + manual review |
| **Driver-rider collusion** | Trip pattern analysis | Both accounts suspended |
| **Account takeover** | New device + location change | Force re-verification |

---

## 3. Location Privacy

```mermaid
flowchart TB
    GPS2["📍 GPS Data (Most sensitive!)"]

    subgraph "Protection Measures"
        P_A["🔐 Encrypted in transit + at rest"]
        P_B["⏰ Retention: Anonymized after 90 days"]
        P_C["🔑 Access: Need-to-know basis<br/>(Strict ACL, audit logged)"]
        P_D["👻 Anonymization: Remove PII for analytics"]
        P_E["🌍 Geo-fencing: Comply with local laws"]
    end

    GPS2 --> P_A & P_B & P_C & P_D & P_E
```

### 2016 Data Breach — Bài Học

| Facts | Details |
|---|---|
| **What** | 57M rider + driver records exposed |
| **How** | Attackers found AWS credentials in GitHub repo |
| **Impact** | $148M settlement, CISO hired |
| **Lessons** | 1) Never commit secrets to git |
|  | 2) Rotate credentials regularly |
|  | 3) Encrypt all PII at rest |
|  | 4) Bug bounty program expanded |

---

## 4. Physical Safety Features

| Feature | Description |
|---|---|
| **Emergency SOS** | One-tap call to emergency services + share location |
| **Trip sharing** | Share real-time trip with trusted contacts |
| **Driver verification** | Background check + photo verification |
| **PIN verification** | Rider gives PIN to correct driver |
| **Audio recording** | Opt-in trip audio recording |
| **RideCheck** | AI detects unusual stops → proactive check-in |

---

## 5. So Sánh Security: Uber vs Others

| Layer | Uber | Netflix | YouTube | WhatsApp |
|---|---|---|---|---|
| **Focus** | Payment fraud + safety | Content piracy | Copyright | Message privacy |
| **Unique** | Mastermind rules engine | DRM watermark | Content ID | Signal E2EE |
| **Threat** | Financial fraud + GPS spoof | Piracy | View fraud | Surveillance |
| **Breach notable** | 2016 (57M records) | Minimal | Minimal | Minimal |
| **Physical safety** | SOS, trip sharing | N/A | N/A | N/A |

---

## Mapping → NestJS

| Pattern | Uber | NestJS Implementation |
|---|---|---|
| **Payment tokenization** | PSP tokens | Stripe SDK + `@nestjs/config` |
| **Rules engine** | Mastermind | `json-rules-engine` npm |
| **ML fraud scoring** | Real-time inference | TensorFlow.js / Python gRPC |
| **Penny-drop** | Random hold verification | Stripe auth holds |
| **Location encryption** | TLS + at-rest | `crypto` module + PostgreSQL pgcrypto |
| **Data retention** | Auto-anonymize | Scheduled BullMQ job |
