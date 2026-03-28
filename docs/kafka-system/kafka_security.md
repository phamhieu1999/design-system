# Apache Kafka - Security Analysis

> Kafka bảo vệ data-in-motion: mTLS, SASL, ACLs, encryption.

---

## Tổng Quan

```mermaid
graph TB
    subgraph "Layer 1: Transport"
        TLS9["🔒 TLS/mTLS<br/>(Encryption in transit)"]
        PLAINTEXT["⚠️ PLAINTEXT<br/>(Dev only!)"]
    end

    subgraph "Layer 2: Authentication"
        SASL_PLAIN["🔑 SASL/PLAIN"]
        SASL_SCRAM["🔑 SASL/SCRAM-SHA"]
        SASL_OAUTH["🔑 SASL/OAUTHBEARER"]
        MTLS2["📜 mTLS (Certificate)"]
    end

    subgraph "Layer 3: Authorization"
        ACL["📋 ACLs<br/>(Per topic/group/cluster)"]
        RBAC3["👑 RBAC<br/>(Confluent Platform)"]
    end

    subgraph "Layer 4: Data"
        ENC_REST["🔐 Encryption at rest<br/>(Disk-level)"]
        SCHEMA2["📋 Schema validation<br/>(Schema Registry)"]
    end
```

---

## 1. Authentication — SASL Mechanisms

| Mechanism | Security | Use Case |
|---|---|---|
| **SASL/PLAIN** | Low (credentials in clear) | Dev/test only |
| **SASL/SCRAM-SHA-256** | Medium (challenge-response) | Production (no KDC) |
| **SASL/SCRAM-SHA-512** | High | Production (recommended) |
| **SASL/GSSAPI (Kerberos)** | High | Enterprise with KDC |
| **SASL/OAUTHBEARER** | High | Cloud-native, OAuth2 |
| **mTLS** | Highest | Zero-trust environments |

---

## 2. ACL Authorization

```mermaid
flowchart TB
    CLIENT2["Client: order-service"] --> AUTHORIZE["🔑 ACL Check"]

    AUTHORIZE --> ALLOW5["✅ ALLOW:<br/>• Read topic 'orders'<br/>• Write topic 'order-events'<br/>• Read group 'order-consumer-group'"]

    AUTHORIZE --> DENY2["🚫 DENY:<br/>• Read topic 'payments' (not authorized)<br/>• Alter cluster config<br/>• Create topics"]

    subgraph "ACL Format"
        FORMAT["Principal: User:order-service<br/>Resource: Topic:orders<br/>Operation: Read<br/>Permission: ALLOW<br/>Host: *"]
    end
```

---

## 3. Data Governance — Schema Registry

```mermaid
flowchart TB
    PRODUCER5["Producer"] --> SCHEMA3["📋 Schema Registry:<br/>Validate schema before produce"]

    SCHEMA3 --> COMPAT{"Compatible?"}
    COMPAT -->|"✅ Yes"| PRODUCE2["Write to Kafka"]
    COMPAT -->|"❌ No"| REJECT["🚫 Reject<br/>(Breaking change!)"]

    subgraph "Compatibility Modes"
        BACK["BACKWARD: Consumer can read old data"]
        FORWARD["FORWARD: Old consumer can read new data"]
        FULL["FULL: Both directions"]
    end

    style REJECT fill:#ff6b6b,color:#fff
```

---

## 4. So Sánh Security

| Layer | Kafka | Stripe | Amazon | Netflix |
|---|---|---|---|---|
| **Transport** | mTLS / TLS | TLS 1.2+ | TLS + VPC | TLS |
| **Auth** | SASL / mTLS | API keys | IAM | OAuth |
| **Authorization** | ACLs / RBAC | Key scoping | IAM policies | Token scope |
| **Data protection** | Disk encryption | HSM vault | KMS | DRM |

---

## Mapping → NestJS

| Pattern | Kafka | NestJS Implementation |
|---|---|---|
| **TLS** | `ssl: { ca, cert, key }` | KafkaJS ssl config |
| **SASL** | `sasl: { mechanism, username, password }` | KafkaJS sasl config |
| **Schema validation** | Avro/Protobuf schemas | `@kafkajs/confluent-schema-registry` |
| **ACLs** | `kafka-acls` CLI | Admin API or Terraform |
