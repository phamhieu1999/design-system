# Amazon - Security Analysis

> Amazon bảo vệ 310M+ accounts, hàng tỷ giao dịch/năm, PCI DSS Level 1.

---

## Tổng Quan

```mermaid
graph TB
    subgraph "Layer 1: Network"
        SHIELD["🌍 AWS Shield (DDoS)"]
        WAF4["🛡️ AWS WAF"]
        CF2["🌍 CloudFront (Edge)"]
    end

    subgraph "Layer 2: Auth"
        AMZN_AUTH["🔑 Amazon Cognito-style"]
        MFA8["📱 MFA (SMS, Auth app)"]
        DEVICE3["📲 Device trust scoring"]
        SESSION4["🎫 Session + IP anomaly"]
    end

    subgraph "Layer 3: Payment"
        PCI2["💳 PCI DSS Level 1"]
        TOKEN4["🔐 Payment tokenization"]
        A2A["🏦 Amazon Pay (A2A)"]
    end

    subgraph "Layer 4: Fraud"
        ML_FRAUD2["🤖 ML Fraud Detection"]
        RULES2["📋 Real-time rules engine"]
        REVIEW_FRAUD["👤 Manual review queue"]
    end

    subgraph "Layer 5: Seller Trust"
        SELLER_VERIFY["✅ Seller verification"]
        PRODUCT_AUTH["🏷️ Product authenticity"]
        A2Z["🛡️ A-to-Z Guarantee"]
    end

    subgraph "Layer 6: Data"
        ENC3["🔐 KMS encryption at rest"]
        IAM["🔑 IAM (least privilege)"]
        AUDIT3["📋 CloudTrail audit"]
    end
```

---

## 1. Payment Security

```mermaid
flowchart TB
    PURCHASE["🛒 Purchase"] --> TOKENIZE2["🔐 Tokenize card<br/>(PCI-compliant vault)"]
    TOKENIZE2 --> AUTH6["🔑 Authorization<br/>(Real-time)"]
    AUTH6 --> FRAUD_CHECK["🤖 Fraud check<br/>(< 100ms)"]
    FRAUD_CHECK -->|"Low risk"| APPROVE["✅ Approve"]
    FRAUD_CHECK -->|"Medium risk"| CHALLENGE2["🔐 Step-up auth<br/>(OTP, re-enter CVV)"]
    FRAUD_CHECK -->|"High risk"| DECLINE["❌ Decline"]

    APPROVE --> CAPTURE["💰 Capture<br/>(At shipment)"]
    
    NOTE10["Auth ≠ Capture:<br/>Amazon authorizes at checkout<br/>but captures only when item ships"]

    style NOTE10 fill:#4c6ef5,color:#fff
```

---

## 2. Fraud Detection

```mermaid
flowchart TB
    ORDER3["🛒 Transaction"] --> SIGNALS

    subgraph "Risk Signals"
        SIG_A["📱 Device fingerprint"]
        SIG_B["📍 IP geolocation"]
        SIG_C["🕐 Time patterns"]
        SIG_D["💳 Payment velocity"]
        SIG_E["📦 Shipping address history"]
        SIG_F["👤 Account age + behavior"]
    end

    SIGNALS --> SIG_A & SIG_B & SIG_C & SIG_D & SIG_E & SIG_F
    SIG_A & SIG_B & SIG_C & SIG_D & SIG_E & SIG_F --> ML8["🤖 ML Risk Score"]
    ML8 --> DECISION2{"Decision (< 100ms)"}

    DECISION2 -->|"Approve"| OK2["✅ Process"]
    DECISION2 -->|"Review"| QUEUE4["👤 Manual review"]
    DECISION2 -->|"Decline"| BLOCK5["🚫 Block"]

    subgraph "Seller Fraud"
        SF1["Fake reviews detection"]
        SF2["Counterfeit product scanning"]
        SF3["Price manipulation alerts"]
    end
```

---

## 3. Marketplace Trust — A-to-Z Guarantee

```mermaid
flowchart TB
    BUYER_ISSUE["😤 Buyer issue"] --> CLAIM["📋 A-to-Z Claim"]
    CLAIM --> AI_REVIEW["🤖 AI Classification"]

    AI_REVIEW --> AUTO_RESOLVE["✅ Auto-resolve<br/>(Clear case, refund)"]
    AI_REVIEW --> SELLER_RESP["📨 Ask seller to respond<br/>(3 days)"]
    SELLER_RESP --> MANUAL["👤 Manual arbitration"]

    subgraph "Seller Consequences"
        SC1["📉 Order Defect Rate (ODR)"]
        SC2["⚠️ Warning → Suspension"]
        SC3["🚫 Account termination"]
    end
```

---

## 4. Data Protection & Compliance

| Category | Implementation |
|---|---|
| **Encryption at rest** | KMS-managed keys (AES-256) |
| **Encryption in transit** | TLS 1.2+ everywhere |
| **Access control** | IAM + least privilege |
| **Audit** | CloudTrail (all API calls logged) |
| **Compliance** | PCI DSS, SOC 2, GDPR, HIPAA |
| **Data isolation** | VPC + security groups |
| **Secret management** | AWS Secrets Manager (auto-rotate) |
| **Key rotation** | Automatic via KMS |

---

## 5. So Sánh Security: Amazon vs Others

| Layer | Amazon | Uber | Netflix | YouTube |
|---|---|---|---|---|
| **Focus** | Payment + marketplace trust | Payment + safety | Content protection | Copyright |
| **Unique** | A-to-Z Guarantee | Mastermind rules | DRM watermark | Content ID |
| **Fraud** | ML + rules (< 100ms) | RADAR anomaly | N/A | View fraud |
| **Compliance** | PCI DSS L1 | PCI DSS | SOC 2 | Google internal |
| **Auth capture** | Split (auth ≠ capture) | Immediate | Subscription | N/A |
| **Cloud** | Own (AWS) | GCP | AWS | Google Cloud |

---

## Mapping → NestJS

| Pattern | Amazon | NestJS Implementation |
|---|---|---|
| **PCI tokenization** | Custom vault | Stripe Elements (never touch cards) |
| **Auth/capture split** | Authorize now, capture later | Stripe `auth → capture` flow |
| **Fraud ML** | Real-time scoring | Stripe Radar / custom ML via gRPC |
| **A-to-Z claims** | Auto-resolve + manual | State machine + BullMQ workers |
| **KMS encryption** | AWS KMS | `@aws-sdk/client-kms` / `crypto` |
| **IAM** | Least privilege | NestJS Guards + CASL `@casl/ability` |
| **Audit trail** | CloudTrail | TypeORM audit columns + event log table |
