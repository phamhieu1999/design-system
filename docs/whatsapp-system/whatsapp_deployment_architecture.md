# WhatsApp - Deployment & Architecture

> WhatsApp phục vụ **2B+ users** với team nhỏ nhờ Erlang/BEAM — xử lý **100B+ messages/ngày**.

---

## 1. Quy Mô

| Metric | Giá trị |
|---|---|
| Monthly Active Users | 2+ tỷ |
| Messages per day | 100+ tỷ |
| Concurrent connections | 1+ tỷ |
| Media shared/day | 7+ tỷ (ảnh, video, audio) |
| Voice/Video calls/day | 2+ tỷ phút |
| Engineers (early days) | ~50 engineers cho 900M users |

---

## 2. Technology Stack

```mermaid
graph TB
    subgraph "Client"
        IOS2["📱 iOS (Swift/ObjC)"]
        ANDROID2["📱 Android (Java/Kotlin)"]
        WEB2["🌐 Web (React)"]
        DESKTOP["🖥️ Desktop (Electron)"]
    end

    subgraph "Protocol"
        XMPP["📡 Modified XMPP<br/>(Binary-encoded, minimal)"]
        NOISE["🔐 Noise Protocol<br/>(Transport encryption)"]
    end

    subgraph "Backend (Erlang/OTP on BEAM)"
        CONN["🔗 Connection Servers<br/>(Millions conn/server)"]
        MSG_SVC["📨 Message Router"]
        PRESENCE["🟢 Presence Service"]
        GROUP["👥 Group Service"]
    end

    subgraph "Data Stores"
        MNESIA["🗄️ Mnesia<br/>(Distributed, in-memory)"]
        MYSQL3["🐬 MySQL<br/>(Persistent storage)"]
        ROCKS["🪨 RocksDB<br/>(Message queue)"]
    end

    subgraph "Infrastructure"
        FREEBSD["🖥️ FreeBSD<br/>(OS - networking perf)"]
        META_DC["🏢 Meta Data Centers"]
    end

    IOS2 & ANDROID2 & WEB2 & DESKTOP --> XMPP --> NOISE --> CONN
    CONN --> MSG_SVC & PRESENCE & GROUP
    MSG_SVC --> MNESIA & MYSQL3 & ROCKS
```

### Tại Sao Erlang?

| Feature | Lợi ích cho WhatsApp |
|---|---|
| **Lightweight processes** | 1 process per connection, millions/server |
| **"Let it crash"** | 1 user crash không ảnh hưởng others |
| **Hot code swap** | Update code KHÔNG cần restart server |
| **OTP Supervision** | Auto-restart crashed processes |
| **Pattern matching** | Xử lý messages routing cực nhanh |
| **Distributed** | Built-in clustering across nodes |

### Tại Sao FreeBSD?

- Networking stack performance vượt trội Linux cho concurrent TCP connections
- Tuned kernel cho millions simultaneous connections per server
- WhatsApp custom-patched cả BEAM VM lẫn FreeBSD kernel

---

## 3. System Architecture

```mermaid
graph TB
    subgraph "Client Layer"
        PHONE["📱 2B+ Devices"]
    end

    subgraph "Edge Layer"
        LB4["⚖️ Load Balancer"]
    end

    subgraph "Connection Layer"
        CS1["🔗 Connection Server 1<br/>(2M+ connections)"]
        CS2["🔗 Connection Server 2"]
        CS3["🔗 Connection Server N"]
    end

    subgraph "Application Layer"
        ROUTER["📨 Message Router<br/>(Erlang process per user)"]
        PRES["🟢 Presence Manager<br/>(Online/Offline/Typing)"]
        GRP["👥 Group Manager<br/>(Fan-out to members)"]
        MEDIA2["📸 Media Service<br/>(Upload/Download)"]
        CALL["📞 Call Signaling<br/>(WebRTC)"]
    end

    subgraph "Storage Layer"
        MNES["🗄️ Mnesia<br/>(Session, routing tables)"]
        OFFLINE["📥 Offline Queue<br/>(RocksDB)"]
        MYSQL4["🐬 MySQL<br/>(Account data)"]
        BLOB3["📦 Blob Storage<br/>(Media, encrypted)"]
    end

    PHONE --> LB4
    LB4 --> CS1 & CS2 & CS3
    CS1 & CS2 & CS3 --> ROUTER & PRES & GRP & MEDIA2 & CALL
    ROUTER --> MNES & OFFLINE
    GRP --> MNES
    MEDIA2 --> BLOB3
```

---

## 4. Deployment & Operations

```mermaid
flowchart TB
    subgraph "Deployment Strategy"
        CODE["👨‍💻 Code Change"] --> TEST2["🧪 Tests (Erlang EUnit)"]
        TEST2 --> BUILD2["📦 Release (OTP Release)"]
        BUILD2 --> ROLLING["🔄 Rolling Deploy<br/>(Node by node)"]
        ROLLING --> HOT["⚡ Hot Code Swap<br/>(No restart needed!)"]
    end

    subgraph "Unique: Zero-downtime Updates"
        HOT --> LIVE["✅ Live system updated<br/>• No disconnections<br/>• No thundering herd<br/>• No message loss"]
    end

    style HOT fill:#51cf66,color:#fff
    style LIVE fill:#4c6ef5,color:#fff
```

**Key insight:** Nhờ Erlang hot code swap, WhatsApp update code mà KHÔNG cần restart servers → không có hiện tượng millions users reconnect cùng lúc (thundering herd).

---

## So Sánh Stack: WhatsApp vs Instagram vs Twitter

| Aspect | WhatsApp | Instagram | Twitter/X |
|---|---|---|---|
| **Language** | Erlang/OTP | Python (Django) | Scala/Java (JVM) |
| **VM/Runtime** | BEAM | CPython | JVM |
| **OS** | FreeBSD | Linux | Linux |
| **Concurrency model** | Actor (1 process/user) | Async (Celery) | Finagle (Futures) |
| **Protocol** | Modified XMPP | HTTP/REST | HTTP/REST |
| **Primary DB** | Mnesia + MySQL | PostgreSQL | Manhattan |
| **Deployment** | Hot code swap | Rolling restart | K8s rolling |
