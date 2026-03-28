# Spotify - Xử Lý Đồng Thời Cao & Audio Streaming

> 600M+ users streaming đồng thời, 100M+ tracks, instant playback < 200ms.

---

## 1. Audio Streaming Architecture

```mermaid
flowchart TB
    USER6["👤 User taps Play"] --> CLIENT3["📱 Client"]
    CLIENT3 --> CACHE_CHECK{"Local cache?"}

    CACHE_CHECK -->|"Cache HIT"| LOCAL["▶️ Play from cache<br/>(Instant!)"]
    CACHE_CHECK -->|"Cache MISS"| CDN8["🌍 CDN Edge"]

    CDN8 --> CDN_CHECK{"CDN has file?"}
    CDN_CHECK -->|"HIT"| STREAM3["📡 Stream from edge"]
    CDN_CHECK -->|"MISS"| ORIGIN2["📦 GCS Origin"]
    ORIGIN2 --> CDN8

    STREAM3 --> BUFFER["📊 Buffer:<br/>Pre-fetch next track<br/>while current plays"]

    style LOCAL fill:#1DB954,color:#fff
    style STREAM3 fill:#51cf66,color:#fff
```

### Audio Codec & Quality

| Tier | Codec | Bitrate | Quality |
|---|---|---|---|
| **Low** | Ogg Vorbis | 24 kbps | Data saver |
| **Normal** | Ogg Vorbis | 96 kbps | Acceptable |
| **High** | Ogg Vorbis | 160 kbps | Good (Free default) |
| **Very High** | Ogg Vorbis | 320 kbps | Premium only |
| **HiFi** | FLAC | 1411 kbps | Lossless (Premium) |

### P2P → CDN Evolution

```mermaid
timeline
    title Streaming Architecture Evolution
    2008 : Hybrid P2P + Server
         : Users share cached audio
         : Reduced server costs 80%
    2014 : Full CDN migration
         : P2P too complex for mobile
         : Google Cloud CDN
    2024 : Edge caching + local cache
         : Predictive pre-fetching
         : Offline mode
```

---

## 2. Adaptive Bitrate

```mermaid
flowchart TB
    MONITOR4["📊 Monitor network"] --> CHECK6{"Bandwidth?"}

    CHECK6 -->|"WiFi (fast)"| HIGH2["320 kbps (Very High)"]
    CHECK6 -->|"4G (good)"| MED2["160 kbps (High)"]
    CHECK6 -->|"3G (slow)"| LOW2["96 kbps (Normal)"]
    CHECK6 -->|"Weak signal"| LOW3["24 kbps (Low)"]

    subgraph "Chunked Delivery"
        CHUNK3["Audio split into ~512KB chunks"]
        RANGE["HTTP Range Requests"]
        SWITCH["Quality switches between chunks"]
    end
```

---

## 3. Offline Mode

```mermaid
flowchart TB
    DOWNLOAD["⬇️ Download playlist"] --> ENCRYPT6["🔐 Encrypt + DRM<br/>(Widevine)"]
    ENCRYPT6 --> LOCAL_STORE["💾 Local storage<br/>(Encrypted files)"]

    subgraph "Smart Downloads"
        SMART["🤖 Auto-download:<br/>• Frequently played songs<br/>• New Discover Weekly<br/>• Daily Mixes updates"]
        REMOVE["🗑️ Auto-remove:<br/>• Songs not played in 30 days<br/>• Storage pressure"]
    end

    OFFLINE_PLAY["✈️ Offline Play"] --> DECRYPT["🔓 Decrypt locally"] --> PLAY["▶️ Play"]
    PLAY --> QUEUE2["📋 Queue play events<br/>(Sync when online)"]
```

---

## 4. Gapless Playback & Crossfade

```mermaid
sequenceDiagram
    participant Player as 🎵 Player
    participant Buffer as 📊 Buffer
    participant CDN9 as CDN

    Player->>Buffer: Playing Track A (3:30)
    
    Note over Player,CDN9: At 3:00 (30s before end)
    Player->>CDN9: Pre-fetch Track B chunks
    CDN9-->>Buffer: Track B loaded

    Note over Player: At 3:28 (crossfade start)
    Player->>Player: Fade out Track A<br/>Fade in Track B
    
    Note over Player: Seamless transition!
    Player->>Player: Track B playing
```

---

## 5. Playback State Sync (Connect)

```mermaid
flowchart TB
    subgraph "Spotify Connect"
        PHONE["📱 Phone<br/>(Controller)"]
        SPEAKER["🔊 Smart Speaker<br/>(Player)"]
        LAPTOP["💻 Laptop"]
        CAR["🚗 Car"]
    end

    PHONE --> CS["☁️ Connect Service<br/>(Real-time state sync)"]
    CS --> SPEAKER & LAPTOP & CAR

    subgraph "Synced State"
        STATE["• Current track + position<br/>• Queue<br/>• Volume<br/>• Shuffle/Repeat"]
    end
```

---

## 6. So Sánh Streaming: Spotify vs Others

| Aspect | Spotify | YouTube Music | Netflix | WhatsApp |
|---|---|---|---|---|
| **Media** | Audio | Audio + Video | Video | Audio messages |
| **Codec** | Ogg Vorbis / FLAC | AAC / Opus | AV1 / VP9 | Opus |
| **CDN** | Google CDN | Google CDN | Open Connect | N/A |
| **Offline** | Encrypted download | Encrypted download | Encrypted download | Auto-download |
| **Latency** | < 200ms start | < 1s start | < 1s start | Real-time |
| **Unique** | Gapless + crossfade | Music videos | Per-title encode | E2EE audio |

---

## Mapping → NestJS

| Pattern | Spotify | NestJS Implementation |
|---|---|---|
| **Audio streaming** | Chunked + CDN | S3 + CloudFront + Range requests |
| **Offline mode** | Encrypted local | `crypto` + client-side storage |
| **Adaptive bitrate** | Quality switching | Multiple encodings + client negotiation |
| **Pre-fetch** | Next-track preload | WebSocket hint from server |
| **Connect (sync)** | Real-time state | `@nestjs/websockets` + Redis Pub/Sub |
| **Play events** | Kafka + batch process | Kafka → queue → batch to ClickHouse |
