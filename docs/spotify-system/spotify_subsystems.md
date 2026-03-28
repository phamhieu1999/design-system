# Spotify - Subsystems Analysis

> Recommendation (Discover Weekly), Search, Podcasts, Wrapped, Data Pipeline.

---

## 1. Recommendation Engine — 3 Pillars

```mermaid
flowchart TB
    subgraph "Pillar 1: Collaborative Filtering"
        CF3["👥 User-Item Matrix"]
        CF3 --> TASTE["Taste Clusters:<br/>'Users like you also listen to...'"]
        CF3 --> IMPLICIT["Implicit signals:<br/>• Streams, saves, skips<br/>• Playlist adds<br/>• Listen duration"]
    end

    subgraph "Pillar 2: Audio Analysis (CNN)"
        AUDIO2["🎵 Raw audio"] --> SPECTRO["Spectrogram<br/>(Time-frequency)"]
        SPECTRO --> CNN["🧠 CNN Model"]
        CNN --> FEATURES2["Features:<br/>• Tempo, key, loudness<br/>• Danceability, energy<br/>• Valence (mood)"]
    end

    subgraph "Pillar 3: NLP"
        TEXT2["📝 Blog posts, reviews,<br/>news articles, metadata"]
        TEXT2 --> NLP2["🧠 NLP Model"]
        NLP2 --> CONTEXT2["Cultural context:<br/>'chill summer vibes'<br/>'workout energy'"]
    end

    TASTE & FEATURES2 & CONTEXT2 --> ENSEMBLE2["🏆 Ensemble Model"]
    ENSEMBLE2 --> OUTPUT

    subgraph "Recommendation Products"
        OUTPUT --> DW["🎧 Discover Weekly (30 songs/Mon)"]
        OUTPUT --> DAILYMIX["📻 Daily Mix (6 mixes)"]
        OUTPUT --> RELEASE["🆕 Release Radar (Fri)"]
        OUTPUT --> HOME2["🏠 Home Page rows"]
    end

    style ENSEMBLE2 fill:#1DB954,color:#fff
```

### Cold Start Problem

| Scenario | Solution |
|---|---|
| **New user** | Ask genres on signup → bootstrap with popular |
| **New track (no listens)** | Audio CNN → similar sounding tracks |
| **New artist** | NLP (blog mentions) + audio similarity |

---

## 2. Search Architecture

```mermaid
flowchart TB
    QUERY4["🔍 'taylor swift'"] --> INTENT2["Intent Detection<br/>• Artist? Track? Podcast?"]
    INTENT2 --> PARALLEL3

    subgraph "Parallel Search"
        S_TRACK["🎵 Tracks"]
        S_ARTIST["👤 Artists"]
        S_ALBUM["💿 Albums"]
        S_PLAYLIST["📋 Playlists"]
        S_PODCAST["🎙️ Podcasts"]
    end

    PARALLEL3 --> S_TRACK & S_ARTIST & S_ALBUM & S_PLAYLIST & S_PODCAST
    S_TRACK & S_ARTIST & S_ALBUM & S_PLAYLIST & S_PODCAST --> MERGE["Merge + Rank<br/>• Popularity<br/>• Personal affinity<br/>• Freshness"]
    MERGE --> RESULTS5["📋 Results"]

    subgraph "Features"
        TYPO2["Fuzzy: 'tayler swif' → Taylor Swift"]
        VOICE3["🎤 Voice search"]
        LYRIC["🎵 'Search by lyrics'"]
    end
```

---

## 3. Spotify Wrapped — Year-End Data Product

```mermaid
flowchart TB
    YEAR["📅 Full year of listening data"] --> PIPELINE3["Pipeline:<br/>BigQuery batch processing"]
    PIPELINE3 --> CALCULATE["Calculate per-user:<br/>• Top artists (5)<br/>• Top songs (100)<br/>• Genres explored<br/>• Minutes listened<br/>• Listening personality<br/>• Top podcasts"]
    CALCULATE --> GENERATE["Generate assets:<br/>• Animated cards<br/>• Shareable images<br/>• Video summaries"]
    GENERATE --> DELIVER["📱 Push to all 600M+ users"]

    subgraph "Engineering Challenge"
        EC1["Pre-compute for 600M users"]
        EC2["Launch simultaneously worldwide"]
        EC3["Handle 10x normal traffic"]
        EC4["Social sharing spike"]
    end

    style DELIVER fill:#1DB954,color:#fff
```

---

## 4. Podcasts & Audiobooks

```mermaid
flowchart TB
    subgraph "Podcast Pipeline"
        RSS["📡 RSS feed ingest"]
        RSS --> TRANSCODE5["🔀 Transcode audio"]
        TRANSCODE5 --> TRANSCRIBE["📝 Auto-transcribe<br/>(Speech-to-text)"]
        TRANSCRIBE --> INDEX2["🔍 Index transcripts<br/>(Searchable!)"]
        INDEX2 --> RECOMMEND3["🤖 Recommend based on:<br/>• Music taste → podcast genres<br/>• Transcript content<br/>• Listening patterns"]
    end

    subgraph "Monetization"
        DYNAMIC_AD["🎯 Dynamic ad insertion<br/>(Personalized per listener)"]
        SPOTIFY_AD["Ad served from Spotify<br/>(Not embedded in audio)"]
    end
```

---

## 5. Data Pipeline

```mermaid
flowchart LR
    subgraph "Events (Billions/day)"
        EV_PLAY["▶️ Play events"]
        EV_SEARCH2["🔍 Search"]
        EV_SKIP["⏭️ Skip events"]
        EV_SAVE["💾 Save/like"]
    end

    EV_PLAY & EV_SEARCH2 & EV_SKIP & EV_SAVE --> KAFKA22["📨 Kafka"]

    subgraph "Real-time"
        KAFKA22 --> BEAM2["⚡ Apache Beam<br/>(Scio / Dataflow)"]
        BEAM2 --> REALTIME2["Real-time:<br/>• Listening activity<br/>• Trending detection"]
    end

    subgraph "Batch"
        KAFKA22 --> BQ2["📊 BigQuery (Data Lake)"]
        BQ2 --> ML11["🤖 Model training"]
        BQ2 --> WRAPPED2["🎁 Wrapped computation"]
        BQ2 --> ROYALTY2["💰 Royalty calculation"]
    end
```

---

## 6. So Sánh Tổng Hợp: 9 Systems

| Dimension | Spotify | Stripe | Amazon | Uber | YouTube | Netflix | Instagram | Twitter | WhatsApp |
|---|---|---|---|---|---|---|---|---|---|
| **Primary** | Audio streaming | Payments | E-commerce | Rides | Video | Streaming | Photo | Microblog | Messaging |
| **Language** | Java/Scala | Ruby/Java | Java | Go/Java | Python/C++ | Java | Python | Scala | Erlang |
| **ML highlight** | 3-pillar rec | Radar fraud | Item-to-Item | ETA | Two-Tower | Everything rec | Explore | Trending | Spam |
| **Open source** | Backstage, Scio | Stripe SDKs | AWS | H3, Jaeger | Vitess, VP9 | Netflix OSS | Fewer | Zipkin | Fewer |
| **Unique** | Audio CNN | Idempotency | Bezos Mandate | DOMA | Content ID | Chaos Eng | TAO | Snowflake | E2EE |

---

## Spotify Unique Innovations

| Innovation | Impact |
|---|---|
| **Backstage** | Developer portal → CNCF project, used by 2000+ companies |
| **Squad Model** | Org design → studied/adopted worldwide |
| **Audio CNN** | Deep learning on raw audio → cold start solved |
| **Discover Weekly** | Personalized playlist → redefined music discovery |
| **Wrapped** | Data product → viral marketing event globally |
| **Scio** | Scala API for Apache Beam → open source |
| **Dynamic Ad Insertion** | Per-listener podcast ads → new monetization model |

---

## Mapping → NestJS

| Subsystem | Spotify | NestJS Implementation |
|---|---|---|
| **Recommendation** | CF + CNN + NLP | TensorFlow.js / Python gRPC |
| **Search** | Custom + fuzzy | `@nestjs/elasticsearch` + fuzzy |
| **Wrapped** | BigQuery batch | ClickHouse + scheduled BullMQ |
| **Podcast ingest** | RSS → transcode | `rss-parser` + `fluent-ffmpeg` |
| **Ad insertion** | Dynamic per-listener | Server-side ad stitching |
| **Backstage** | CNCF developer portal | Directly adopt Backstage! |
| **Data pipeline** | Kafka → Beam → BigQuery | Kafka → workers → ClickHouse |
