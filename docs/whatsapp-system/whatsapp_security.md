# WhatsApp - Security Analysis

> WhatsApp là benchmark cho E2EE messaging — Signal Protocol bảo vệ 100B+ messages/ngày.

---

## Tổng Quan: Defense in Depth

```mermaid
graph TB
    subgraph "Layer 1: Transport"
        NOISE2["🔒 Noise Protocol<br/>(Client ↔ Server)"]
        TLS3["🔒 TLS 1.3<br/>(Server ↔ Server)"]
    end

    subgraph "Layer 2: End-to-End Encryption"
        SIGNAL["🔐 Signal Protocol"]
        X3DH["🤝 X3DH Key Exchange"]
        RATCHET["🔄 Double Ratchet"]
    end

    subgraph "Layer 3: Authentication"
        PHONE["📱 Phone Number Verification"]
        MFA3["🔑 2FA (PIN)"]
        DEVICE["📲 Device Verification"]
    end

    subgraph "Layer 4: Data Protection"
        NO_STORE["🗑️ No server-side message storage"]
        BACKUP_ENC["💾 Encrypted Backups"]
        DISAPPEAR["⏰ Disappearing Messages"]
    end

    subgraph "Layer 5: Anti-Abuse"
        SPAM2["🚫 Spam Detection"]
        FORWARD["📤 Forward Limit (5 chats)"]
        REPORT["🚩 Report & Block"]
    end

    NOISE2 --> TLS3 --> SIGNAL --> X3DH --> RATCHET --> PHONE --> MFA3 --> DEVICE --> NO_STORE --> BACKUP_ENC --> DISAPPEAR --> SPAM2 --> FORWARD --> REPORT
```

---

## 1. Signal Protocol — E2E Encryption

### 1.1 Key Exchange (X3DH)

```mermaid
sequenceDiagram
    actor Alice as 👤 Alice
    participant Server as WhatsApp Server
    actor Bob as 👤 Bob

    Note over Bob,Server: Bob registers: uploads PreKey Bundle
    Bob->>Server: Upload: Identity Key + Signed PreKey + One-time PreKeys

    Note over Alice: Alice wants to message Bob (first time)
    Alice->>Server: Request Bob's PreKey Bundle
    Server-->>Alice: Bob's public keys

    Alice->>Alice: X3DH Key Agreement:<br/>• DH(Alice_identity, Bob_signed_prekey)<br/>• DH(Alice_ephemeral, Bob_identity)<br/>• DH(Alice_ephemeral, Bob_signed_prekey)<br/>• DH(Alice_ephemeral, Bob_onetime_prekey)

    Alice->>Alice: Derive shared secret → Root Key
    Alice->>Server: Send encrypted message + Alice's ephemeral public key
    Server->>Bob: Forward (server CANNOT decrypt)

    Bob->>Bob: Perform same X3DH → derive same Root Key
    Bob->>Bob: Decrypt message ✅
```

### 1.2 Double Ratchet Algorithm

```mermaid
flowchart TB
    subgraph "Forward Secrecy"
        MSG1["Message 1: Key K1"] --> KDF1["KDF(K1)"]
        KDF1 --> MSG2["Message 2: Key K2"]
        MSG2 --> KDF2["KDF(K2)"]
        KDF2 --> MSG3["Message 3: Key K3"]
        
        NOTE1["K1 deleted after use<br/>→ Cannot decrypt Msg 1<br/>even if K3 is compromised"]
    end

    subgraph "Post-Compromise Security"
        DH1["DH Ratchet Step 1<br/>(New DH key pair)"]
        DH1 --> DH2["DH Ratchet Step 2<br/>(New shared secret)"]
        DH2 --> HEAL["🔄 Security healed<br/>Attacker locked out"]
    end

    style NOTE1 fill:#51cf66,color:#fff
    style HEAL fill:#4c6ef5,color:#fff
```

| Property | Giải thích |
|---|---|
| **Forward Secrecy** | Compromise key hiện tại → KHÔNG decrypt được messages cũ |
| **Post-Compromise Security** | Sau khi attacker mất access → encryption tự heal |
| **Per-message keys** | Mỗi message một key riêng → compromise 1 ≠ compromise all |
| **AES-256-CBC** | Symmetric encryption cho message content |
| **HMAC-SHA256** | Authentication/integrity check |

---

## 2. Transport Security — Noise Protocol

```mermaid
sequenceDiagram
    participant Client as 📱 WhatsApp Client
    participant Server as 🖥️ WhatsApp Server

    Note over Client,Server: Noise Protocol Handshake (replaces TLS)
    Client->>Server: Client Hello + Ephemeral Key
    Server-->>Client: Server Ephemeral Key + Encrypted payload
    Client->>Server: Encrypted handshake finish

    Note over Client,Server: All subsequent data encrypted<br/>Server authenticated by static key<br/>Client authenticated by account credentials

    loop Message Exchange
        Client->>Server: Encrypted data (Noise frame)
        Server->>Client: Encrypted data (Noise frame)
    end
```

**Tại sao Noise thay vì TLS?** Noise nhẹ hơn, ít round-trips hơn, tối ưu cho mobile (tiết kiệm battery + bandwidth).

---

## 3. Authentication & Registration

```mermaid
flowchart TB
    REG["📱 Register"] --> PHONE2["Enter phone number"]
    PHONE2 --> OTP["📨 SMS/Call OTP"]
    OTP --> VERIFY["✅ Verify OTP"]
    VERIFY --> KEYGEN["🔑 Generate:<br/>• Identity Key Pair<br/>• Signed PreKey<br/>• 100 One-time PreKeys"]
    KEYGEN --> UPLOAD["📤 Upload public keys to server"]
    UPLOAD --> READY["✅ Account ready"]

    subgraph "2FA (Optional)"
        PIN["🔢 6-digit PIN"]
        PIN --> ENCRYPT_PIN["Stored as:<br/>argon2(PIN + salt)"]
        ENCRYPT_PIN --> REQUIRED["Required on new device registration"]
    end

    VERIFY --> PIN
```

---

## 4. Data Protection — Zero Knowledge

```mermaid
graph TB
    subgraph "What Server CAN See (Metadata)"
        M1["📱 Phone numbers (sender/receiver)"]
        M2["⏰ Timestamps"]
        M3["📊 Message size"]
        M4["📍 IP address"]
        M5["📱 Device info"]
    end

    subgraph "What Server CANNOT See (E2EE)"
        E1["💬 Message content"]
        E2["📸 Media content"]
        E3["📞 Call audio/video"]
        E4["📍 Shared location"]
        E5["📎 Attachments"]
    end

    subgraph "Server Storage Policy"
        S_POLICY["Messages: Deleted after delivery<br/>Offline queue: Max 30 days<br/>Media: Temporary (download link expires)"]
    end

    style E1 fill:#51cf66,color:#fff
    style E2 fill:#51cf66,color:#fff
    style M1 fill:#ffd43b,color:#333
```

### Encrypted Backups

```mermaid
flowchart LR
    CHAT["💬 Chat History"] --> ENCRYPT2["🔐 Encrypt with<br/>user-generated password<br/>or 64-digit key"]
    ENCRYPT2 --> CLOUD["☁️ iCloud / Google Drive<br/>(Encrypted blob)"]

    CLOUD --> RESTORE["📱 New device"]
    RESTORE --> DECRYPT["🔓 Decrypt with<br/>password/key"]
    
    NOTE2["WhatsApp & Apple/Google<br/>CANNOT access backup<br/>without user's password"]

    style NOTE2 fill:#51cf66,color:#fff
```

---

## 5. Anti-Abuse & Spam Prevention

```mermaid
flowchart TB
    subgraph "Without Reading Content (E2EE)"
        AB1["📊 Metadata analysis<br/>(sending rate, patterns)"]
        AB2["📱 Device fingerprinting"]
        AB3["🔗 Link preview via client<br/>(Not server-side)"]
        AB4["📤 Forward count tracking<br/>(Limit: 5 chats)"]
        AB5["🚩 User reports<br/>(Client sends decrypted msg)"]
    end

    AB1 & AB2 & AB3 & AB4 & AB5 --> ML3["🤖 ML Classifier"]
    ML3 --> ACTION{"Action"}
    ACTION -->|"Spam"| BAN["🚫 Ban account"]
    ACTION -->|"Suspicious"| LIMIT["⚖️ Rate limit"]
    ACTION -->|"Normal"| ALLOW2["✅ Allow"]

    style BAN fill:#ff6b6b,color:#fff
```

**Challenge:** Vì E2EE, WhatsApp KHÔNG đọc được nội dung → chỉ detect spam qua metadata + user reports.

---

## 6. So Sánh Security: WhatsApp vs Instagram vs Twitter

| Layer | WhatsApp | Instagram | Twitter/X |
|---|---|---|---|
| **E2E Encryption** | ✅ Signal Protocol (default) | ❌ Removed (May 2026) | ❌ None |
| **Transport** | Noise Protocol | TLS 1.3 | TLS 1.3 |
| **Server storage** | Temp only (delete after delivery) | Permanent | Permanent |
| **Auth** | Phone + OTP + 2FA PIN | OAuth 2.0 + 2FA | OAuth 2.0 + 2FA |
| **Metadata** | Collected by Meta | Collected by Meta | Collected by X |
| **Content moderation** | Client-side reports only | Server-side AI | Server-side AI + Community Notes |
| **Forward secrecy** | ✅ Per-message keys | ❌ | ❌ |
| **Backup encryption** | ✅ User-controlled | ❌ | ❌ |

---

## Mapping → NestJS

| Pattern | WhatsApp | NestJS Implementation |
|---|---|---|
| **E2E Encryption** | Signal Protocol | `libsignal-protocol-javascript` |
| **Key Exchange** | X3DH | `@noble/curves` (Curve25519) |
| **Symmetric Encryption** | AES-256-CBC | `crypto` module (Node.js built-in) |
| **Transport** | Noise Protocol | TLS + `@nestjs/websockets` |
| **OTP Auth** | SMS/Call verification | Twilio + `speakeasy` (TOTP) |
| **2FA PIN** | Argon2 hashed | `argon2` npm package |
| **Forward Limit** | Track forward count | Redis counter per message_id |
| **Encrypted Backup** | User-key encrypted | `crypto.createCipher` + user passphrase |

> [!IMPORTANT]
> **Bài học #1 từ WhatsApp:** E2EE messaging có thể hoạt động ở quy mô 2B+ users. Trade-off: server KHÔNG THỂ moderate content → phải dựa vào metadata analysis + user reports.
