# WhatsApp - Subsystems Analysis

> Voice/Video calls, Status/Stories, Multi-device, Payments, Reliability.

---

## 1. Voice & Video Calls (WebRTC)

```mermaid
sequenceDiagram
    actor Alice as 👤 Alice
    participant Server as Signaling Server
    participant TURN as TURN/STUN Server
    actor Bob as 👤 Bob

    Alice->>Server: Call request (encrypted signaling)
    Server->>Bob: Incoming call notification

    Bob->>Server: Accept call
    
    par ICE Candidate Exchange
        Alice->>Server: ICE candidates (network info)
        Server->>Bob: Forward candidates
        Bob->>Server: ICE candidates
        Server->>Alice: Forward candidates
    end

    alt Direct P2P possible
        Alice<-->Bob: Direct P2P connection<br/>(E2E encrypted, lowest latency)
    else NAT/Firewall blocks P2P
        Alice<-->TURN: Relay through TURN server
        TURN<-->Bob: Relay through TURN server
        Note over TURN: TURN relays encrypted packets<br/>(Cannot see content)
    end
```

### Group Calls

```mermaid
graph TB
    subgraph "Group Call (up to 32 participants)"
        SFU["SFU Server<br/>(Selective Forwarding Unit)"]
        
        U1["👤 User 1"] <-->|"Upload 1 stream"| SFU
        U2["👤 User 2"] <-->|"Upload 1 stream"| SFU
        U3["👤 User 3"] <-->|"Upload 1 stream"| SFU
        U4["👤 User 4"] <-->|"Upload 1 stream"| SFU
        
        SFU -->|"Download N-1 streams"| U1 & U2 & U3 & U4
    end

    NOTE3["Each participant uploads 1 stream<br/>Downloads N-1 streams<br/>SFU forwards (does NOT transcode)<br/>All streams E2E encrypted"]
```

---

## 2. Multi-Device Architecture

```mermaid
flowchart TB
    subgraph "Primary Device (Phone)"
        PHONE3["📱 Phone<br/>• Source of truth<br/>• Identity keys<br/>• Message history"]
    end

    subgraph "Companion Devices"
        WEB3["🌐 WhatsApp Web"]
        DESKTOP2["🖥️ Desktop App"]
        TABLET["📱 Tablet (future)"]
    end

    subgraph "How it works"
        QR["📷 QR Code scan<br/>→ Establish encrypted link"]
        SYNC["🔄 Message sync<br/>using companion protocol"]
        INDEPENDENT["Each device has own<br/>encryption keys"]
    end

    PHONE3 --> QR --> WEB3 & DESKTOP2 & TABLET
    PHONE3 --> SYNC

    NOTE4["Since 2021: Companion devices<br/>can work independently<br/>(phone can be offline)"]

    style PHONE3 fill:#4c6ef5,color:#fff
```

**Key change (2021+):** Trước đây Web phải proxy qua Phone. Giờ mỗi device có own encryption keys → hoạt động độc lập.

---

## 3. Status/Stories System

```mermaid
flowchart TB
    POST3["📸 User posts Status"] --> ENCRYPT3["🔐 Encrypt with<br/>Sender Key (per contact)"]
    ENCRYPT3 --> UPLOAD2["📤 Upload encrypted media"]
    UPLOAD2 --> NOTIFY["📨 Notify contacts"]

    NOTIFY --> ONLINE3["Online contacts:<br/>Push immediately"]
    NOTIFY --> OFFLINE3["Offline contacts:<br/>Queue notification"]

    subgraph "Status Lifecycle"
        TTL["⏰ 24-hour TTL"]
        TTL --> AUTO_DEL["🗑️ Auto-delete from<br/>server + all viewers"]
    end
```

---

## 4. WhatsApp Payments (India)

```mermaid
sequenceDiagram
    actor Sender as 👤 Sender
    participant WA as WhatsApp
    participant UPI as UPI (NPCI)
    participant Bank_S as Sender's Bank
    participant Bank_R as Receiver's Bank
    actor Receiver as 👤 Receiver

    Sender->>WA: Send ₹100 to Receiver
    WA->>UPI: Payment request (VPA)
    UPI->>Bank_S: Debit ₹100
    Bank_S-->>UPI: ✅ Debited
    UPI->>Bank_R: Credit ₹100
    Bank_R-->>UPI: ✅ Credited
    UPI-->>WA: Transaction success
    WA->>Sender: ✅ Payment sent
    WA->>Receiver: ✅ ₹100 received
```

---

## 5. Reliability & SRE

```mermaid
flowchart TB
    subgraph "Erlang Fault Tolerance"
        crash["❌ Process crash"] --> supervisor["Supervisor detects"]
        supervisor --> restart["🔄 Restart process<br/>(microseconds)"]
        restart --> resume["✅ User reconnects<br/>(transparent)"]
    end

    subgraph "Zero-downtime Deployment"
        update["📦 Code update"] --> hot_swap["⚡ Hot code swap"]
        hot_swap --> no_restart["✅ No server restart<br/>No user disconnection"]
    end

    subgraph "Data Durability"
        msg_in["Message received"] --> replicate["Replicate to N nodes"]
        replicate --> ack_server["ACK only after replication"]
        ack_server --> durable["✅ Message won't be lost"]
    end
```

### Uptime & SLAs

| Metric | Target |
|---|---|
| Availability | 99.99% (< 53 min downtime/year) |
| Message delivery | < 200ms (same region) |
| Offline delivery | Within seconds of reconnect |
| Call setup | < 3 seconds |

---

## 6. So Sánh Tổng Hợp: WhatsApp vs Instagram vs Twitter

| Dimension | WhatsApp | Instagram | Twitter/X |
|---|---|---|---|
| **Primary** | Messaging | Photo/Video sharing | Microblogging |
| **Language** | Erlang/OTP | Python/Django | Scala/Java |
| **Concurrency** | Actor model (BEAM) | Async (Celery) | Futures (Finagle) |
| **Encryption** | E2EE (Signal) | TLS only | TLS only |
| **Server storage** | Temporary (transit) | Permanent | Permanent |
| **Protocol** | XMPP (modified) | HTTP/REST | HTTP/REST |
| **Fan-out** | Group fan-out (< 1024) | Hybrid push/pull | Hybrid push/pull |
| **Unique tech** | BEAM hot swap | TAO social graph | Snowflake IDs |
| **Key innovation** | E2EE at 2B scale | Media pipeline | Real-time search |

---

## Mapping → NestJS

| Subsystem | WhatsApp | NestJS Implementation |
|---|---|---|
| **Voice/Video** | WebRTC + TURN | `mediasoup` or `livekit` + `@nestjs/websockets` |
| **Multi-device** | Companion protocol | Device registry in Redis + per-device keys |
| **Status/Stories** | 24h TTL + E2E | Redis `SET` with TTL + Bull delayed job for cleanup |
| **Payments** | UPI integration | Stripe/payment gateway + `@nestjs/microservices` |
| **Fault tolerance** | Erlang supervisor | PM2 cluster + K8s liveness probes |
| **Hot deploy** | BEAM hot swap | K8s rolling update (closest equivalent) |
