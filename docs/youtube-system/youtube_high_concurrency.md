# YouTube - Xử Lý Đồng Thời Cao & Video Pipeline

> 500+ giờ video uploaded/phút, 1B+ giờ xem/ngày.

---

## 1. Video Upload & Processing Pipeline

```mermaid
flowchart TB
    UPLOAD4["🎬 Creator uploads"] --> CHUNK2["📦 Chunked Upload<br/>(10-50MB chunks, resumable)"]
    CHUNK2 --> GCS_RAW["📦 GCS (Raw file)"]
    GCS_RAW --> PUBSUB2["📨 Pub/Sub: video.uploaded"]

    PUBSUB2 --> DAG2["🔀 DAG Scheduler"]

    subgraph "Parallel Processing (1000s workers)"
        DAG2 --> VALIDATE["✅ Validate<br/>(format, codec, duration)"]
        DAG2 --> MODERATE["🤖 AI Moderation<br/>(NSFW, copyright, violence)"]
        DAG2 --> TRANSCODE4["⚡ Transcode<br/>(Split by keyframes)"]
        DAG2 --> THUMB["🖼️ Thumbnails<br/>(Auto-generate + AI select)"]
        DAG2 --> CAPTION["📝 Auto-captions<br/>(Speech-to-text)"]
        DAG2 --> AUDIO["🎵 Audio fingerprint<br/>(Content ID)"]
    end

    subgraph "Transcode Output"
        TRANSCODE4 --> R_144["144p"]
        TRANSCODE4 --> R_360["360p"]
        TRANSCODE4 --> R_720["720p"]
        TRANSCODE4 --> R_1080["1080p"]
        TRANSCODE4 --> R_4K["4K"]
        TRANSCODE4 --> R_AV1["AV1 (modern)"]
        TRANSCODE4 --> R_VP9["VP9 (YouTube default)"]
    end

    R_144 & R_360 & R_720 & R_1080 & R_4K --> PACKAGE2["📦 DASH/HLS Segments"]
    PACKAGE2 --> CDN_FILL["📤 Push to CDN edges"]

    style TRANSCODE4 fill:#e64980,color:#fff
    style CDN_FILL fill:#51cf66,color:#fff
```

### Codec Strategy

| Codec | Quality/Efficiency | Support | Use |
|---|---|---|---|
| **H.264** | Baseline | Universal | Legacy devices |
| **VP9** | 30-50% smaller than H.264 | Most browsers | YouTube default |
| **AV1** | 30% smaller than VP9 | Growing | Premium 4K, Shorts |
| **H.265** | Good compression | Limited (patents) | Apple devices |

**Progressive processing:** SD xong trước → HD later → 4K last. User có thể xem ngay ở 360p trong khi 4K đang encode.

---

## 2. CDN & Global Distribution

```mermaid
graph TB
    subgraph "Google Global Edge Network"
        CORE["🏢 Core DCs<br/>(Origin, processing)"]
        POP1["🌍 PoP Americas"]
        POP2["🌍 PoP Europe"]
        POP3["🌍 PoP Asia"]

        CORE --> POP1 & POP2 & POP3

        POP1 --> ISP1["📡 ISP Edge Cache"]
        POP2 --> ISP2["📡 ISP Edge Cache"]
        POP3 --> ISP3["📡 ISP Edge Cache"]
    end

    subgraph "Cache Hit Flow"
        USER5["👤 Viewer"] --> ISP_HIT["ISP Cache<br/>(Cache HIT 95%+)"]
        ISP_HIT --> USER5
        NOTE9["Most video NEVER touches<br/>core datacenters"]
    end

    style ISP_HIT fill:#51cf66,color:#fff
```

### YouTube CDN vs Netflix Open Connect

| Aspect | YouTube (Google CDN) | Netflix (Open Connect) |
|---|---|---|
| **Ownership** | Google-owned edge network | Custom appliances in ISPs |
| **Model** | Google-operated PoPs | ISP-hosted, Netflix-managed |
| **Content** | UGC (unpredictable popularity) | Licensed (predictable) |
| **Caching** | Reactive + predictive | Proactive (fill off-peak) |
| **Scale** | billions of unique videos | ~15K titles |
| **Challenge** | Long tail (rare videos) | High bitrate (4K, Dolby) |

---

## 3. Adaptive Bitrate Streaming

```mermaid
sequenceDiagram
    participant Client as 📱 YouTube Player
    participant CDN as 🌍 CDN Edge

    Client->>CDN: GET manifest.mpd (DASH)
    CDN-->>Client: Manifest (all quality levels)

    loop Every 2-5 second segment
        Client->>Client: Measure: bandwidth, buffer, device
        
        alt First load (cold start)
            Client->>CDN: Low quality (fast start)
            Note over Client: Start playing < 1s
        else Buffer healthy + good bandwidth
            Client->>CDN: Higher quality segment
        else Buffer low
            Client->>CDN: Lower quality segment
        end
        
        CDN-->>Client: Video segment
        Client->>Client: Decode & render
    end
```

---

## 4. View Count & Engagement at Scale

```mermaid
flowchart TB
    VIEW["👁️ Video viewed"] --> KAFKA15["Pub/Sub event"]

    subgraph "Real-time Path (< 1s)"
        KAFKA15 --> MEMCOUNT["In-memory counter<br/>(Approximate, fast)"]
        MEMCOUNT --> DISPLAY["📊 Display count<br/>(May lag slightly)"]
    end

    subgraph "Accurate Path (batched)"
        KAFKA15 --> VALIDATION["🤖 View Validation<br/>• Bot detection<br/>• Duplicate detection<br/>• Minimum watch time<br/>• Geographic anomaly"]
        VALIDATION --> BIGTABLE2["📊 Bigtable<br/>(Authoritative count)"]
        BIGTABLE2 --> SYNC["🔄 Sync to display"]
    end

    subgraph "Monetization Count"
        VALIDATION --> AUDIT["💰 Audited count<br/>(For ad revenue split)"]
    end

    style VALIDATION fill:#ff6b6b,color:#fff
```

**View validation:** YouTube does NOT count every "view". Bot views, too-short views, duplicate views are filtered → accurate count essential for **ad revenue sharing** with creators.

---

## 5. Live Streaming Architecture

```mermaid
flowchart TB
    STREAMER2["🎥 Creator starts live"] --> INGEST2["Ingest Server<br/>(RTMP/WebRTC)"]
    INGEST2 --> REALTIME_TRANSCODE["⚡ Real-time Transcode<br/>(Low-latency, multiple qualities)"]
    REALTIME_TRANSCODE --> SEGMENT3["HLS/DASH Segments<br/>(2-4s chunks)"]
    SEGMENT3 --> CDN5["🌍 CDN Push"]

    CDN5 --> VIEWERS["👁️ Millions of viewers"]

    subgraph "Live Chat"
        CHAT["💬 Chat"] --> PUBSUB3["Pub/Sub"]
        PUBSUB3 --> MODERATE2["🤖 Real-time moderation"]
        MODERATE2 --> BROADCAST["📡 Broadcast to viewers"]
    end

    subgraph "Latency Modes"
        ULTRA["⚡ Ultra-low: < 2s<br/>(WebRTC, fewer qualities)"]
        LOW["🏃 Low: 3-5s<br/>(Reduced manifest)"]
        NORMAL["📺 Normal: 10-15s<br/>(Full ABR, most stable)"]
    end
```

---

## Mapping → NestJS

| Pattern | YouTube | NestJS Implementation |
|---|---|---|
| **Chunked upload** | GCS resumable upload | `multer` + S3 multipart upload |
| **DAG pipeline** | Custom scheduler | BullMQ job dependencies |
| **Transcode** | FFmpeg on C++ workers | `fluent-ffmpeg` + BullMQ workers |
| **View counting** | Pub/Sub → validation → Bigtable | Kafka → validation service → ClickHouse |
| **ABR streaming** | DASH/HLS | `hls.js` + nginx-rtmp-module |
| **Live streaming** | RTMP → HLS | mediasoup / LiveKit |
| **Content ID** | Audio fingerprint ML | `chromaprint` + custom matching |
