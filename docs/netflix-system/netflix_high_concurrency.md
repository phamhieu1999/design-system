# Netflix - Xử Lý Đồng Thời Cao & Streaming

> 500M+ giờ streaming/ngày, ~15% bandwidth internet toàn cầu.

---

## 1. Video Encoding Pipeline

```mermaid
flowchart TB
    SOURCE["🎬 Source File (Mezzanine)<br/>4K, 10-bit, ProRes"] --> ANALYZE["🤖 Content Analysis<br/>(Per-title complexity)"]
    
    ANALYZE --> LADDER["📊 Custom Encoding Ladder<br/>(Per-title/per-shot)"]

    LADDER --> CHUNK["✂️ Split into chunks"]
    CHUNK --> PAR["⚡ Parallel Encoding<br/>(1000s EC2 instances)"]
    
    PAR --> R1["240p @ 200kbps"]
    PAR --> R2["480p @ 750kbps"]
    PAR --> R3["720p @ 2.5Mbps"]
    PAR --> R4["1080p @ 5Mbps"]
    PAR --> R5["4K HDR @ 16Mbps"]
    
    R1 & R2 & R3 & R4 & R5 --> VMAF["📏 VMAF Quality Check<br/>(Perceptual quality score)"]
    
    VMAF --> PACKAGE["📦 Package:<br/>• DASH/HLS segments<br/>• Multiple audio tracks<br/>• Subtitles"]
    
    PACKAGE --> S3["📦 S3 Storage"]
    S3 --> FILL["📤 Fill OCAs<br/>(Off-peak hours)"]

    style PAR fill:#4c6ef5,color:#fff
    style VMAF fill:#51cf66,color:#fff
```

### Per-Title Encoding — Netflix Innovation

| Content Type | Approach | Example |
|---|---|---|
| Animation (simple) | Lower bitrate, same quality | 1080p @ 1.5 Mbps |
| Drama (medium) | Standard ladder | 1080p @ 4 Mbps |
| Action (complex) | Higher bitrate needed | 1080p @ 8 Mbps |

**VMAF** (Video Multi-Method Assessment Fusion): Netflix-created quality metric → thay PSNR/SSIM → correlate tốt hơn với human perception.

---

## 2. Open Connect — Netflix CDN

```mermaid
graph TB
    subgraph "Traditional CDN"
        TRAD_USER["User"] --> TRAD_ISP["ISP"] --> INTERNET["🌐 Public Internet"] --> TRAD_CDN["CDN Edge"] --> TRAD_ORIGIN["Origin Server"]
    end

    subgraph "Netflix Open Connect"
        OC_USER["User"] --> OC_ISP["ISP"]
        OC_ISP --> OCA_LOCAL["📦 OCA<br/>(Inside ISP datacenter!)"]
        OCA_LOCAL --> OC_USER
        NOTE5["Video NEVER crosses<br/>public internet"]
    end

    style OCA_LOCAL fill:#e64980,color:#fff
    style NOTE5 fill:#51cf66,color:#fff
```

### OCA (Open Connect Appliance)

| Spec | Value |
|---|---|
| **Storage** | 100+ TB NVMe SSDs |
| **Throughput** | 90+ Gbps per appliance |
| **Cost to ISP** | Free (Netflix provides) |
| **Benefit to ISP** | Reduces transit costs 60-95% |
| **Locations** | 17,000+ servers, 6,000+ ISP/IXP locations |
| **Traffic** | >95% of Netflix traffic served from OCAs |

### Proactive Cache Filling

```mermaid
flowchart LR
    subgraph "Off-peak (2AM-6AM)"
        S3_3["S3 (Origin)"] --> FILL2["📤 Push popular content<br/>to regional OCAs"]
        FILL2 --> PREDICT["🤖 Predictive model:<br/>• New releases<br/>• Trending content<br/>• Regional preferences"]
    end

    subgraph "Peak hours"
        USER3["📱 User presses Play"] --> OCA_HIT["✅ Cache HIT (99%+)"]
        OCA_HIT --> STREAM2["🎬 Stream from local OCA<br/>(< 1 hop away)"]
    end
```

---

## 3. Adaptive Bitrate Streaming (ABR)

```mermaid
sequenceDiagram
    participant Client as 📱 Netflix Client
    participant OCA as 📦 OCA (Local)

    Client->>OCA: Request manifest.mpd (DASH)
    OCA-->>Client: Manifest (all quality levels + segments)

    loop Every 2-4 second segment
        Client->>Client: Monitor bandwidth + buffer
        
        alt Good bandwidth (20 Mbps)
            Client->>OCA: GET segment_42_1080p.mp4
        else Bandwidth drops (3 Mbps)
            Client->>OCA: GET segment_43_480p.mp4
            Note over Client: Seamless quality switch
        else Bandwidth recovers
            Client->>OCA: GET segment_44_720p.mp4
        end
        
        OCA-->>Client: Video segment
    end
```

---

## 4. Resilience Patterns — "Design for Failure"

```mermaid
flowchart TB
    subgraph "Circuit Breaker (Resilience4j)"
        REQ4["Request"] --> CB3{"Circuit state?"}
        CB3 -->|"CLOSED (healthy)"| CALL["Call remote service"]
        CB3 -->|"OPEN (failing)"| FALLBACK2["Return fallback<br/>(cached/default data)"]
        CB3 -->|"HALF-OPEN (testing)"| TEST3["Try 1 request"]
        
        CALL -->|"Success"| OK["✅ Return response"]
        CALL -->|"Failure (> threshold)"| OPEN2["🔴 Open circuit<br/>(60s cooldown)"]
        TEST3 -->|"Success"| CLOSE2["🟢 Close circuit"]
        TEST3 -->|"Failure"| KEEP["🔴 Keep open"]
    end

    subgraph "Bulkhead Pattern"
        POOL_A["Thread Pool A (50)<br/>Recommendation Service"]
        POOL_B["Thread Pool B (30)<br/>Billing Service"]
        POOL_C["Thread Pool C (20)<br/>Search Service"]
        NOTE6["Pool A exhausted ≠<br/>Pool B/C affected"]
    end

    style FALLBACK2 fill:#ffd43b,color:#333
    style OPEN2 fill:#ff6b6b,color:#fff
```

### Chaos Engineering — Simian Army

```mermaid
flowchart TB
    subgraph "The Simian Army"
        CM["🐒 Chaos Monkey<br/>Kill random instances"]
        CG["🦍 Chaos Gorilla<br/>Kill entire AZ"]
        CK["🔧 Chaos Kong<br/>Kill entire region"]
        LM["⏰ Latency Monkey<br/>Inject network delays"]
        DM["💀 Doctor Monkey<br/>Health checks"]
        JM["🧹 Janitor Monkey<br/>Clean unused resources"]
    end

    subgraph "Purpose"
        CM --> VERIFY3["Verify: Auto-scaling works"]
        CG --> VERIFY4["Verify: AZ failover works"]
        CK --> VERIFY5["Verify: Region failover works"]
    end

    style CM fill:#ff6b6b,color:#fff
    style CG fill:#e64980,color:#fff
    style CK fill:#862e9c,color:#fff
```

### Graceful Degradation Tiers

| Tier | Trigger | Action |
|---|---|---|
| **Normal** | All healthy | Full personalization |
| **Degraded L1** | Recommendation down | Show generic "popular" lists |
| **Degraded L2** | Search down | Disable search, show browse only |
| **Degraded L3** | Auth degraded | Allow cached sessions |
| **Emergency** | Major outage | Static fallback page |

---

## 5. So Sánh Concurrency: Netflix vs Others

| Aspect | Netflix | Instagram | Twitter | WhatsApp |
|---|---|---|---|---|
| **Challenge** | Massive bandwidth | Write amplification | Read amplification | Connection count |
| **Peak metric** | 500M+ hrs/day | 2B+ likes/day | 200B+ timeline views/day | 100B+ msgs/day |
| **Solution** | Own CDN (Open Connect) | Hybrid fan-out | Hybrid fan-out | BEAM actor model |
| **Resilience** | Chaos Engineering | Redundancy | Priority queues | Supervision trees |
| **Unique** | Per-title encoding | Sharded counters | Trending detection | Hot code swap |

---

## Mapping → NestJS

| Pattern | Netflix | NestJS Implementation |
|---|---|---|
| **Circuit Breaker** | Resilience4j | `opossum` npm package |
| **Bulkhead** | Thread pool isolation | Worker threads / `piscina` |
| **Fallback** | Cached/default response | `@Catch()` exception filter + Redis cache |
| **Chaos Engineering** | Chaos Monkey | `chaos-mesh` (K8s) / custom middleware |
| **ABR Streaming** | DASH/HLS | `hls.js` (client) + video segmentation pipeline |
| **CDN** | Open Connect | CloudFront / Cloudflare |
| **Canary Deploy** | Spinnaker ACA | ArgoCD progressive delivery |
