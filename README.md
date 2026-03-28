# 🏗️ Scalable System Design Architecture

> Phân tích chuyên sâu kiến trúc của 13 hệ thống công nghệ lớn nhất thế giới để rút ra các **Design Patterns** cốt lõi và hướng dẫn **áp dụng vào thực tế với NestJS Microservices**.

---

## 🎯 Mục Tiêu Dự Án

Dự án này là kho tàng kiến trúc (Architecture Repository) phân tích cách các gã khổng lồ công nghệ giải quyết bài toán:
- Xử lý đồng thời cực lớn (High Concurrency / Scale).
- Đảm bảo tính sẵn sàng cao và chống gián đoạn (Resilience / High Availability).
- Bảo mật hệ thống và dữ liệu (Security).
- Thiết kế luồng dữ liệu và Storage tối ưu (Data Pipeline & Storage).

Mỗi hệ thống được phân tích sẽ đi kèm với **Mapping Pattern**, hướng dẫn cách implement những khái niệm khổng lồ đó vào một backend hiện đại sử dụng **Node.js, NestJS, Kafka, Redis, PostgreSQL**.

---

## 📚 Master Index: Các Hệ Thống Đã Phân Tích (12/13)

### Tier 1: Social & Real-time (Nền Tảng Giao Tiếp)
| Hệ Thống | Trọng Tâm Kiến Trúc | Link |
|---|---|---|
| **Instagram** | Quản lý Media, Storage sharding, Fan-out newsfeed | [docs/instagram-system](./docs/instagram-system) |
| **Twitter / X** | Real-time trending, Timeline fan-out, Event Sourcing | [docs/twitter-system](./docs/twitter-system) |
| **WhatsApp** | End-to-End Encryption (Signal Protocol), Erlang concurrency | [docs/whatsapp-system](./docs/whatsapp-system) |
| **URL Shortener**| Base62 Hashing, Read-heavy optimization, Analytics | [docs/url-shortener-system](./docs/url-shortener-system) |

### Tier 2: Media & Streaming (Xử Lý Audio/Video)
| Hệ Thống | Trọng Tâm Kiến Trúc | Link |
|---|---|---|
| **YouTube** | Video transcoding pipeline, Adaptive Bitrate (ABR) | [docs/youtube-system](./docs/youtube-system) |
| **Netflix** | Microservices gôc, Global CDN, Chaos Engineering | [docs/netflix-system](./docs/netflix-system) |
| **Spotify** | Audio streaming, Collaborative filtering, Squad model | [docs/spotify-system](./docs/spotify-system) |

### Tier 3: E-commerce, Fintech & Mobility (Giao Dịch Phức Tạp)
| Hệ Thống | Trọng Tâm Kiến Trúc | Link |
|---|---|---|
| **Amazon** | Distributed checkout, Chaotic storage, Bezos API mandate | [docs/amazon-system](./docs/amazon-system) |
| **Stripe** | Idempotency, Double-entry ledger, Radar Fraud ML | [docs/stripe-system](./docs/stripe-system) |
| **Uber** | Geo-spatial matching, Real-time tracking, DOMA | [docs/uber-system](./docs/uber-system) |

### Tier 4: Core Infrastructure & Big Data (Hạ Tầng Lõi)
| Hệ Thống | Trọng Tâm Kiến Trúc | Link |
|---|---|---|
| **Google Search**| Inverted index, PageRank, Spanner/Colossus, MapReduce | [docs/google-search-system](./docs/google-search-system) |
| **Apache Kafka** | Distributed log, Exactly-once (EOS), KRaft consensus | [docs/kafka-system](./docs/kafka-system) |
| **Redis** | In-memory structures, Sentinel HA, Cluster mode | [docs/redis-system](./docs/redis-system) |

---

## 🛠 Cấu Trúc Mỗi Phân Tích

Mỗi thư mục hệ thống bên trong `docs/` đều tuân theo chuẩn 4 tài liệu sau (Standardized Format):

1. **`<system>_deployment_architecture.md`**: Tech stack, Evolution, Deployment topology, Data models.
2. **`<system>_high_concurrency.md`**: Scale strategies, Caching, DB Sharding, Event pipelines.
3. **`<system>_security.md`**: Auth/Z, Fraud detection, Encryption, Threat modeling.
4. **`<system>_subsystems.md`**: Đi sâu vào các features làm nên tên tuổi (VD: Recommend, Ads, Checkout) và so sánh chéo.

---

## 🧠 Key NestJS Implementation Patterns Kết Tinh

Sau khi phân tích hàng loạt hệ thống trên, đây là bộ cẩm nang (Cheat Sheet) cho hệ thống NestJS:

- **Idempotency (Stripe/Amazon)**: Sử dụng Interceptor/Guard + Redis `SETNX` với Transaction ID để tránh Double-charging.
- **Saga Pattern (Uber/Stripe)**: Thay vì 2PC Database, dùng Event Choreography với Kafka để rollback Distributed Transactions.
- **Fan-out on Write (Twitter/Instagram)**: Push tweets/posts vào Redis Timeline Cache của Active Followers bằng BullMQ Background Jobs.
- **Event Sourcing (Kafka/Twitter)**: View Database (Read) là kết quả aggregate từ Kafka Log (Write) → CQRS Pattern.
- **Geo-Spatial (Uber)**: Sử dụng PostgreSQL/PostGIS (H3 Index) thay vì query tọa độ vòng lặp tốn kém.
- **Chunked Media (YouTube/Spotify)**: Video Streaming không gửi cả file mà dùng HTTP `206 Partial Content` (Range Requests).

---

> *Bản quyền tài liệu thuộc về thư viện kiến trúc thiết kế nội bộ. Cập nhật lần cuối: Tháng 03/2026.*
