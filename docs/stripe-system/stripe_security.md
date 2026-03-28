# Stripe - Security Analysis

> PCI DSS Level 1 Service Provider, $60B+ fraud blocked/năm bởi Radar.

---

## Tổng Quan

```mermaid
graph TB
    subgraph "Layer 1: Network"
        TLS7["🔒 TLS 1.2+ everywhere"]
        CERT_PIN["📌 Certificate pinning (mobile)"]
    end

    subgraph "Layer 2: PCI DSS Level 1"
        SCOPE["Merchant scope reduction"]
        SAQ["SAQ A (lowest burden for merchants)"]
        AUDIT5["Annual audit by QSA"]
    end

    subgraph "Layer 3: Tokenization"
        VAULT3["🔐 Token Vault<br/>(Isolate PANs from all systems)"]
        HSM3["🔑 HSM<br/>(Hardware key management)"]
    end

    subgraph "Layer 4: Fraud (Radar)"
        RADAR2["🤖 Radar<br/>(1000+ signals, < 100ms)"]
        RULES3["📋 Custom rules"]
        THREE_DS["🔐 3D Secure (SCA)"]
    end

    subgraph "Layer 5: Access Control"
        RBAC2["👑 API key scoping"]
        RESTRICTED["🔒 Restricted keys"]
        CONNECT["🔗 Connect (platform permissions)"]
    end

    subgraph "Layer 6: Monitoring"
        ANOMALY2["🚨 Anomaly detection"]
        WEBHOOK_SIG["✍️ Webhook signatures"]
        AUDIT6["📋 Request logging"]
    end
```

---

## 1. PCI DSS — How Stripe Reduces Merchant Burden

```mermaid
flowchart TB
    subgraph "Without Stripe"
        W1["Merchant server handles card data"]
        W2["Full PCI DSS audit required"]
        W3["👎 Costly, complex"]
    end

    subgraph "With Stripe Elements"
        S1["Card data → Stripe Elements (iframe)"]
        S2["Data goes directly to Stripe servers"]
        S3["Merchant NEVER sees card numbers"]
        S4["👍 SAQ-A: simplest compliance"]
    end

    style S4 fill:#51cf66,color:#fff
    style W3 fill:#ff6b6b,color:#fff
```

---

## 2. Tokenization Architecture

```mermaid
flowchart TB
    CARD2["💳 Card: 4242 4242 4242 4242"] --> ELEMENTS2["Stripe Elements (iframe)"]
    ELEMENTS2 --> STRIPE_SRV["Stripe PCI Environment"]
    STRIPE_SRV --> ENCRYPT5["🔐 Encrypt with HSM key"]
    ENCRYPT5 --> VAULT4["🗝️ Token Vault<br/>Store: encrypted PAN"]
    VAULT4 --> TOKEN6["Return: tok_1234abcd"]

    TOKEN6 --> MERCHANT2["Merchant stores token<br/>(Useless if stolen!)"]

    subgraph "On Payment"
        MERCHANT2 --> CHARGE2["POST /charges {token: tok_1234}"]
        CHARGE2 --> VAULT4
        VAULT4 --> DECRYPT2["Decrypt → submit to network"]
    end

    style VAULT4 fill:#e64980,color:#fff
```

---

## 3. Stripe Radar — ML Fraud Detection

```mermaid
flowchart TB
    TXN["💳 Transaction"] --> SIGNALS2["1000+ Risk Signals"]

    subgraph "Signal Categories"
        SIG_CARD["💳 Card signals:<br/>• Country mismatch<br/>• Velocity (too many charges)<br/>• Previous chargebacks"]
        SIG_DEVICE["📱 Device signals:<br/>• Browser fingerprint<br/>• IP reputation<br/>• Proxy/VPN detection"]
        SIG_BEHAVIOR["👤 Behavioral:<br/>• Time on page<br/>• Typing patterns<br/>• Mouse movements"]
        SIG_NETWORK["🌐 Network signals:<br/>• Stripe network data<br/>(Billions of data points<br/>across all merchants)"]
    end

    SIGNALS2 --> SIG_CARD & SIG_DEVICE & SIG_BEHAVIOR & SIG_NETWORK
    SIG_CARD & SIG_DEVICE & SIG_BEHAVIOR & SIG_NETWORK --> ML9["🤖 ML Model<br/>(< 100ms scoring)"]

    ML9 --> SCORE4{"Risk Score"}
    SCORE4 -->|"Low (0-20)"| ALLOW4["✅ Allow"]
    SCORE4 -->|"Medium (20-65)"| REVIEW2["🔍 Review (merchant rules)"]
    SCORE4 -->|"High (65-100)"| BLOCK6["🚫 Block"]

    subgraph "Radar Tiers"
        R_FREE["Radar (free): Basic ML protection"]
        R_PRO["Radar for Fraud Teams: Custom rules + insights"]
    end

    style SIG_NETWORK fill:#4c6ef5,color:#fff
```

**Key advantage:** Stripe sees transactions across ALL merchants → can detect cross-merchant fraud patterns that individual merchants cannot.

---

## 4. 3D Secure / Strong Customer Authentication

```mermaid
sequenceDiagram
    participant Merchant as 🏢 Merchant
    participant Stripe as Stripe
    participant Bank as 🏦 Issuing Bank
    participant Customer2 as 👤 Customer

    Merchant->>Stripe: Confirm PaymentIntent
    Stripe->>Stripe: Evaluate SCA requirement<br/>(EU mandate, risk level)

    alt SCA Required
        Stripe-->>Merchant: requires_action
        Merchant->>Customer2: Show 3DS challenge (iframe)
        Customer2->>Bank: Authenticate (OTP, biometric)
        Bank-->>Customer2: ✅ Authenticated
        Customer2->>Stripe: Authentication result
        Stripe->>Stripe: Complete payment
    else SCA Not Required (low risk exemption)
        Stripe->>Stripe: Complete payment directly
    end
```

---

## 5. API Key Security

| Key Type | Scope | Use |
|---|---|---|
| **Publishable key** | Client-safe, tokenize only | `pk_live_xxx` in frontend |
| **Secret key** | Full API access | `sk_live_xxx` server-only |
| **Restricted key** | Limited permissions | Specific endpoints only |
| **Webhook signing secret** | Verify webhook origin | `whsec_xxx` |

---

## 6. So Sánh Security: Stripe vs Others

| Layer | Stripe | Amazon | Uber | Netflix |
|---|---|---|---|---|
| **Focus** | Payment security | Marketplace trust | Safety + fraud | Content protection |
| **Compliance** | PCI DSS Level 1 SP | PCI DSS Level 1 | PCI DSS | SOC 2 |
| **Fraud** | Radar ML (network-wide) | ML scoring | Mastermind rules | N/A |
| **Unique** | Cross-merchant intelligence | A-to-Z Guarantee | GPS spoof detection | DRM watermark |
| **Tokenization** | Vault + HSM | PSP tokenization | PSP tokenization | N/A |

---

## Mapping → NestJS

| Pattern | Stripe | NestJS Implementation |
|---|---|---|
| **PCI scope reduction** | Elements (iframe) | Stripe Elements SDK |
| **Tokenization** | Vault + HSM | Stripe API (never handle cards) |
| **Radar** | Network-wide ML | Stripe Radar (built-in) |
| **3D Secure** | Automatic SCA | Stripe PaymentIntents API |
| **API key scoping** | Restricted keys | Custom API key middleware + CASL |
| **Webhook verification** | HMAC-SHA256 | `stripe.webhooks.constructEvent()` |
