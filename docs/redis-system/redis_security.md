# Redis - Security & Memory Analysis

> Bảo vệ dữ liệu trong RAM: Eviction Policies, Memory footprint, Access Control.

---

## Tổng Quan

```mermaid
graph TB
    subgraph "Layer 1: Network & Auth"
        TLS10["🔒 TLS Support<br/>(Encryption in transit)"]
        AUTH8["🔑 requirepass / ACLs<br/>(Redis 6+)"]
        BIP["🌐 Bind IP Restrictions"]
    end

    subgraph "Layer 2: Commands"
        RENAME["🚧 RENAME_COMMAND<br/>(Hide FLUSHALL, CONFIG)"]
        LUA["📜 LUA Script Sandboxing"]
    end

    subgraph "Layer 3: Memory Security"
        MAX_MEM["📏 maxmemory Limit"]
        EVICTION["🗑️ Eviction Policies<br/>(LRU, LFU, Random)"]
        OOM["🔥 OOM Protection"]
    end
```

---

## 1. Access Control Lists (ACLs) — Từ Redis 6+

Trước Redis 6, mọi client kết nối bằng 1 tài khoản mặc định vạn năng (requirepass).
Từ Redis 6, Redis có ACLs.

```mermaid
flowchart TB
    AUTH9["User: 'report_worker'"] --> ACL2["ACL Check"]

    ACL2 --> ALLOW6["✅ ALLOW:<br/>• GET, MGET, HSCAN<br/>• Keys: cache:*"]

    ACL2 --> DENY3["🚫 DENY:<br/>• Keys: user:password:*<br/>• Commands: FLUSHDB, KEYS, CONFIG"]

    subgraph "ACL Rule Example"
        CMD["ACL SETUSER report_worker on >pass123 ~cache:* +get +mget -flushdb"]
    end
```

---

## 2. Bảo Vệ Dữ Liệu Khi Hết RAM (Eviction Policies)

Khi Redis đạt tới `maxmemory`, nó phải chọn dữ liệu để ném đi. Thuật toán chọn là then chốt.

```mermaid
flowchart TB
    MEM_FULL["🚨 Memory reaches maxmemory"] --> POLICY{"Policy Config?"}

    POLICY --> NO_EVICT["noeviction:<br/>Return error (OOM)<br/>(Chỉ dùng khi Redis làm DB chính)"]
    POLICY --> ALLKEYS["allkeys-*:<br/>Chạm vào toàn bộ Dataset"]
    POLICY --> VOLATILE["volatile-*:<br/>Chỉ chạm các key có set TTL/Expire"]

    subgraph "Algorithms"
        LRU2["LRU (Least Recently Used):<br/>Xóa key lâu nhất không ai đọc"]
        LFU2["LFU (Least Frequently Used):<br/>Xóa key ít lượt truy cập nhất"]
        TTL3["TTL:<br/>Xóa key sắp hết hạn nhất"]
        RANDOM["Random:<br/>Xóa ngẫu nhiên (hiếm dùng)"]
    end
```

### So sánh LRU và LFU
- **LRU:** Chỉ quan tâm mốc thời gian truy cập T. 1 key rác vừa được query cách đây 1s có thể đánh bật 1 key hot query cách đây 5 phút (nhưng trước đó query 1 triệu lần).
- **LFU:** Quan tâm TẦN SUẤT. Key được hit nhiều lần → Giữ lại. LFU phù hợp cho Cache hit rate cao nhất.

---

## 3. Khóa Lệnh Nguy Hiểm

Một số lệnh có độ phức tạp `O(N)` trên Single-threaded engine → Nguy hiểm cực độ:
1. `KEYS *`: Khóa cứng Redis để scan vài triệu keys → Crash ứng dụng (Sử dụng `SCAN` thay thế).
2. `FLUSHALL`: Xóa trắng database.
3. `FLUSHDB`.

**Giải pháp:** Dùng `rename-command` trong redis.conf.
```
rename-command KEYS ""
rename-command FLUSHALL "SECURE_FLUSH_ALL_1234" 
```

---

## So Sánh Memory Management (Eviction)

| Database | Eviction Focus | LRU vs LFU | Storage |
|---|---|---|---|
| **Redis** | Native, siêu nhanh (Approximate LRU) | Hỗ trợ cả 2 | In-memory RAM |
| **Memcached**| Slab allocator eviction | LRU (multi-tier) | In-memory RAM |
| **Cassandra**| N/A (Lưu disk, TTL dựa trên Compaction) | N/A | Disk (SSTable) |
| **Kafka** | Log cleanup (Retention by Time/Size) | N/A | Disk (Sequential) |

---

## Mapping → NestJS

| Pattern | NestJS Implementation |
|---|---|
| **Auth** | Truyền URL: `redis://username:password@host:port` trong ioredis. |
| **Eviction config** | Cầu hình server-side `maxmemory 2gb`, `maxmemory-policy allkeys-lfu`. |
| **O(N) commands** | Cấm dùng `keys *` trong code Nest, thay bằng stream của `scanStream()`. |
| **TTL Default** | Luôn set TTL khi `cacheManager.set(key, val, ttl)`. |
