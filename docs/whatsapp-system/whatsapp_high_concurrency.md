# WhatsApp - Xử Lý Đồng Thời Cao & Message Delivery

> 100B+ messages/ngày, 2M+ connections per server, guaranteed delivery.

---

## 1. Message Delivery Flow

```mermaid
sequenceDiagram
    actor A as 👤 Sender (Alice)
    participant CS_A as Connection Server A
    participant Router as Message Router
    participant Mnesia as Mnesia (Session Store)
    participant Queue as Offline Queue
    participant CS_B as Connection Server B
    participant Push as Push Gateway
    actor B as 👤 Receiver (Bob)

    A->>A: Encrypt (Signal Protocol)
    A->>CS_A: Send encrypted message

    CS_A->>Router: Route message
    Router->>Mnesia: Lookup Bob's connection server

    alt Bob is ONLINE
        Mnesia-->>Router: Bob → Connection Server B
        Router->>CS_B: Forward message
        CS_B->>B: Push via persistent connection
        B->>B: Decrypt message
        B->>CS_B: ACK (delivered)
        CS_B->>Router: Relay ACK
        Router->>CS_A: Relay ACK
        CS_A->>A: ✅✅ Double tick (delivered)
    else Bob is OFFLINE
        Mnesia-->>Router: Bob offline
        Router->>Queue: Store in offline queue
        Router->>CS_A: ✅ Single tick (server received)
        Router->>Push: Trigger push notification
        Push->>B: 🔔 "New message from Alice"

        Note over B,Queue: Later, Bob comes online...
        B->>CS_B: Reconnect
        CS_B->>Queue: Fetch pending messages
        Queue-->>CS_B: Deliver queued messages
        CS_B->>B: Push all pending
        B->>CS_B: ACK (delivered)
        CS_B->>CS_A: ✅✅ Double tick
    end

    Note over A,B: Bob reads the message...
    B->>CS_B: Read receipt
    CS_B->>CS_A: Relay read receipt
    CS_A->>A: ✅✅ Blue ticks (read)
```

### Delivery Guarantees — The Checkmark System

| Status | Symbol | Meaning | Trigger |
|---|---|---|---|
| **Sent** | ✅ | Server received & queued | Server ACK |
| **Delivered** | ✅✅ | Recipient device received | Client ACK |
| **Read** | 🔵🔵 | Recipient opened chat | Client read event |

---

## 2. Connection Management — Millions Per Server

```mermaid
flowchart TB
    subgraph "1 Physical Server"
        BEAM2["BEAM VM<br/>(Custom-patched)"]
        
        BEAM2 --> P1["Process 1<br/>(User #1)"]
        BEAM2 --> P2["Process 2<br/>(User #2)"]
        BEAM2 --> P3["Process 3<br/>(User #3)"]
        BEAM2 --> PN["...<br/>(2M+ processes)"]
    end

    subgraph "Each Process"
        PROC["Erlang Process<br/>• ~2KB memory<br/>• Own mailbox<br/>• Own state<br/>• Isolated heap"]
    end

    subgraph "Supervision Tree"
        SUP["Supervisor"]
        SUP --> W1["Worker 1"]
        SUP --> W2["Worker 2"]
        SUP --> W3["Worker 3 ❌ crash"]
        W3 --> RESTART["🔄 Auto-restart<br/>(no impact on others)"]
    end

    style PN fill:#4c6ef5,color:#fff
    style RESTART fill:#51cf66,color:#fff
```

| Metric | Value |
|---|---|
| Memory per process | ~2-4 KB |
| Processes per server | 2M+ |
| Message latency | < 100ms (same region) |
| Context switch cost | Microseconds (not OS threads!) |

---

## 3. Group Messaging Fan-out

```mermaid
sequenceDiagram
    actor Sender as 👤 Sender
    participant Server as WhatsApp Server
    participant M1 as 👤 Member 1
    participant M2 as 👤 Member 2
    participant M3 as 👤 Member 3 (offline)
    participant Queue2 as Offline Queue

    Sender->>Server: Send to Group (1 message)
    
    par Parallel Fan-out (Erlang spawn)
        Server->>M1: Deliver (encrypted with M1's key)
        Server->>M2: Deliver (encrypted with M2's key)
        Server->>Queue2: Queue for M3 (offline)
    end

    M1-->>Server: ACK ✅✅
    M2-->>Server: ACK ✅✅

    Note over Sender,Queue2: Group uses Sender Key:<br/>Sender encrypts once per group,<br/>each member decrypts with shared key
```

**Sender Key optimization:** Thay vì encrypt N lần cho N members, sender encrypt 1 lần với group key → tất cả members decrypt bằng shared key → giảm compute N lần.

---

## 4. Store-and-Forward Architecture

```mermaid
flowchart TB
    MSG_IN["📨 Incoming Message"] --> STORE{"Recipient online?"}
    
    STORE -->|"Online"| DIRECT["⚡ Direct delivery<br/>(< 100ms)"]
    STORE -->|"Offline"| QUEUE3["📥 Offline Queue<br/>(Encrypted, temporary)"]
    
    QUEUE3 --> PUSH3["🔔 Push Notification<br/>(APNs/FCM)"]
    QUEUE3 --> WAIT["⏳ Wait for reconnect"]
    WAIT --> RECONNECT["📱 User comes online"]
    RECONNECT --> FLUSH["📤 Flush queue<br/>(FIFO order)"]
    FLUSH --> DELETE["🗑️ Delete from server<br/>(No long-term storage!)"]
    
    DIRECT --> DELETE

    style DIRECT fill:#51cf66,color:#fff
    style DELETE fill:#4c6ef5,color:#fff
```

> **Key principle:** Messages are **NOT stored permanently** on WhatsApp servers. Server chỉ là transit point. Sau khi delivered → xóa khỏi server.

---

## 5. Presence & Typing Indicators

```mermaid
flowchart LR
    USER_ACTION["User opens chat"] --> CLIENT["Client sends:<br/>• last_seen update<br/>• typing indicator"]
    CLIENT --> SERVER2["Server routes<br/>to contact's process"]
    SERVER2 --> CONTACT["Contact receives:<br/>• 'Online' / 'Last seen'<br/>• 'Typing...'"]

    subgraph "Optimizations"
        OPT1["Throttle: Max 1 update/10s"]
        OPT2["Only send to visible chats"]
        OPT3["Auto-expire: 'Typing' timeout 15s"]
    end
```

---

## 6. Handling Traffic Spikes (New Year's Eve)

WhatsApp xử lý **75B+ messages** vào đêm giao thừa — peak gấp 2-3x ngày thường.

```mermaid
flowchart TB
    SPIKE2["📈 New Year Spike<br/>(3x normal traffic)"] --> STRATEGIES

    subgraph "Strategies"
        S_A["1️⃣ Pre-provision capacity<br/>(Scale up trước event)"]
        S_B["2️⃣ Message queuing<br/>(Buffer bursts in queue)"]
        S_C["3️⃣ Priority scheduling<br/>(Text > media during peak)"]
        S_D["4️⃣ Graceful degradation<br/>(Delay 'last seen', typing)"]
        S_E["5️⃣ No restart during peak<br/>(Erlang hot swap)"]
    end

    STRATEGIES --> S_A & S_B & S_C & S_D & S_E
```

---

## Mapping → NestJS

| Pattern | WhatsApp | NestJS Implementation |
|---|---|---|
| **Persistent connections** | XMPP over TCP | `@nestjs/websockets` + Socket.IO |
| **1 process per user** | Erlang process | Worker pool + Redis session store |
| **Offline queue** | Mnesia/RocksDB | Bull queue + Redis |
| **ACK system** | Custom protocol | Socket.IO `acknowledgements` |
| **Presence** | Erlang distributed | Redis pub/sub + `SET` with TTL |
| **Group fan-out** | Erlang spawn parallel | BullMQ fan-out job |
| **Push notifications** | APNs/FCM | `firebase-admin` + `node-apn` |
