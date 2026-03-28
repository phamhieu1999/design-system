# Google Search - Subsystems Analysis

> Knowledge Graph, Ads, Verticals, AI Evolution, PageRank.

---

## 1. PageRank — The Original Innovation

```mermaid
flowchart TB
    subgraph "Link Graph"
        A2["Page A"] -->|"links to"| B3["Page B"]
        A2 -->|"links to"| C2["Page C"]
        B3 -->|"links to"| C2
        D2["Page D"] -->|"links to"| A2
        E3["Page E"] -->|"links to"| A2
        E3 -->|"links to"| B3
    end

    subgraph "PageRank Calculation"
        FORMULA["PR(A) = (1-d) + d × Σ PR(Ti)/C(Ti)<br/><br/>d = damping factor (0.85)<br/>Ti = pages linking to A<br/>C(Ti) = outbound links of Ti"]
    end

    subgraph "Intuition"
        INT1["A page is important if<br/>important pages link to it"]
        INT2["More quality inbound links<br/>= higher PageRank"]
        INT3["Iterative algorithm<br/>(converges after ~50 iterations)"]
    end

    style FORMULA fill:#4285F4,color:#fff
```

---

## 2. Knowledge Graph

```mermaid
flowchart TB
    QUERY7["🔍 'Albert Einstein'"] --> KG["🧠 Knowledge Graph<br/>(5B+ entities, 500B+ facts)"]

    KG --> PANEL["Knowledge Panel:<br/>• Born: March 14, 1879<br/>• Nationality: German → American<br/>• Known for: Theory of Relativity<br/>• Nobel Prize: 1921<br/>• Spouse: Mileva Marić, Elsa Einstein"]

    subgraph "Knowledge Sources"
        KS1["📚 Wikipedia / Wikidata"]
        KS2["📊 CIA World Factbook"]
        KS3["🌐 Structured data (Schema.org)"]
        KS4["📝 Google's proprietary data"]
    end

    subgraph "Entity Relations (RDF-like)"
        TRIPLE["Subject → Predicate → Object<br/>Einstein → birthDate → 1879-03-14<br/>Einstein → field → Physics"]
    end
```

---

## 3. Google Ads — The Business Engine

```mermaid
flowchart TB
    SEARCH2["🔍 Search query"] --> AUCTION2["🏦 Real-time Ad Auction<br/>(< 10ms, every query)"]

    AUCTION2 --> FACTORS

    subgraph "Ad Rank Formula"
        FACTORS --> BID["💰 Max CPC Bid"]
        FACTORS --> QS["📊 Quality Score:<br/>• Expected CTR<br/>• Ad relevance<br/>• Landing page quality"]
        FACTORS --> CONTEXT3["📍 Context:<br/>• Location, device<br/>• Time of day<br/>• Other ads"]
    end

    BID & QS & CONTEXT3 --> RANK8["Ad Rank = Bid × Quality × Context"]
    RANK8 --> POSITION2["📢 Ad position (1-4 top)"]
    POSITION2 --> PRICE["💵 Actual CPC:<br/>= Rank below ÷ Your quality + $0.01"]

    style PRICE fill:#34A853,color:#fff
```

**Second-price auction:** You pay just enough to beat the next advertiser → incentivizes truthful bidding.

---

## 4. Search Verticals

```mermaid
flowchart TB
    QUERY8["🔍 User query"] --> INTENT3["🧠 Intent Classification"]

    INTENT3 --> WEB2["🌐 Web results"]
    INTENT3 --> IMAGE["📸 Images"]
    INTENT3 --> VIDEO2["📹 Videos"]
    INTENT3 --> NEWS2["📰 News"]
    INTENT3 --> MAPS2["🗺️ Maps/Local"]
    INTENT3 --> SHOPPING["🛒 Shopping"]

    subgraph "Universal Search"
        BLEND3["Blend verticals into one SERP:<br/>• News carousel<br/>• Map pack (3 local results)<br/>• Image grid<br/>• Video carousel<br/>• Shopping ads"]
    end
```

---

## 5. AI Evolution in Search

```mermaid
timeline
    title Google Search AI Evolution
    2013 : Hummingbird
         : Semantic understanding
    2015 : RankBrain
         : ML for ambiguous queries
    2018 : Neural Matching
         : Concept-level matching
    2019 : BERT
         : Deep NLP, context understanding
    2021 : MUM
         : Multimodal, multilingual AI
    2023 : SGE (Search Generative Experience)
         : AI-generated summaries
    2024 : AI Overviews
         : Gemini-powered answers
    2025 : AI Mode
         : Conversational search
```

---

## 6. So Sánh Tổng Hợp: All 11 Systems

| Dimension | Google Search | Spotify | Stripe | Amazon | Uber | YouTube | Netflix | Instagram | Twitter | WhatsApp | URL Short |
|---|---|---|---|---|---|---|---|---|---|---|---|
| **Core** | Information retrieval | Audio streaming | Payments | E-commerce | Rides | Video | Streaming | Photo social | Microblog | Messaging | Redirect |
| **Scale** | 8.5B queries/d | 600M MAU | $1T/y vol | 310M users | 130M users | 2B MAU | 260M subs | 2B MAU | 550M MAU | 2B MAU | 28B clicks/m |
| **Key DS** | Inverted index | Audio CNN emb. | Double-entry | DynamoDB | H3 hex grid | Vitess | Cassandra | TAO graph | Snowflake | Mnesia | Base62 |
| **Open** | MapReduce, K8s | Backstage | SDKs | AWS | H3, Jaeger | Vitess | Netflix OSS | Fewer | Zipkin | Fewer | N/A |

---

## Google Search Unique Innovations

| Innovation | Impact |
|---|---|
| **PageRank** | Graph-based authority → transformed web search |
| **MapReduce** | Distributed batch processing → industry standard |
| **GFS / Colossus** | Distributed FS → inspired HDFS |
| **Bigtable** | Wide-column NoSQL → inspired HBase, Cassandra |
| **Spanner + TrueTime** | Globally consistent DB with atomic clocks |
| **Borg → Kubernetes** | Container orchestration → industry standard |
| **BERT / MUM** | Deep NLP for search → transformed NLP field |
| **Safe Browsing** | Privacy-preserving malware detection → 4B devices |

---

## Mapping → NestJS

| Subsystem | Google | NestJS Implementation |
|---|---|---|
| **Inverted index** | Custom | Elasticsearch + `@nestjs/elasticsearch` |
| **Knowledge Graph** | Custom graph DB | Neo4j + `neo4j-driver` |
| **Ad auction** | Real-time bidding | Custom auction service + Redis |
| **Verticals** | Intent → route | NestJS microservices per vertical |
| **AI ranking** | BERT/MUM | TensorFlow.js / OpenAI API |
| **Crawler** | Googlebot | `crawlee` + BullMQ |
| **PageRank** | Iterative graph algo | Batch job on graph DB |
