# Redis - Deployment & Architecture

> Bộ nhớ đệm (Cache) và In-Memory Data Store phổ biến nhất thế giới.

---

## 1. Single-Threaded Architecture

```mermaid
flowchart TB
    subgraph "Clients (10,000+ connections)"
        C1["Client 1"]
        C2["Client 2"]
        CN["Client N"]
    end

    subgraph "Redis Server Node"
        EPOLL["OS kernel: epoll / kqueue<br/>(I/O Multiplexing)"]
        
        subgraph "Main Thread"
            EVENT_LOOP["Event Loop (ae.c)"]
            EXEC["Command Execution<br/>(Atomic, no locks!)"]
        end
        
        subgraph "Background Threads (I/O, I/O bound)"
            IO_READ["I/O Read (Parse network)"]
            IO_WRITE["I/O Write (Send response)"]
            UNLINK["Lazy Free (UNLINK)"]
            FSYNC["fsync (AOF)"]
        end
    end

    C1 & C2 & CN --> EPOLL
    EPOLL --> EVENT_LOOP
    EVENT_LOOP --> IO_READ
    IO_READ -.-> EVENT_LOOP
    EVENT_LOOP --> EXEC
    EXEC --> IO_WRITE
    IO_WRITE -.-> C1 & C2 & CN
    
    EXEC -.-> UNLINK & FSYNC

    style EXEC fill:#e03131,color:#fff
```

### Tại sao lại là Single-Threaded?
1. **CPU không phải là cổ chai:** Redis chạy trên RAM → cổ chai thường là Network hoặc Memory B/W.
2. **Không tốn chi phí Context Switching:** Chuyển đổi ngữ cảnh giữa các thread rất đắt.
3. **Không cần Locks/Mutex:** Lập trình nhàn hơn, không bị race condition, mọi lệnh đều tự động là Atomic.

*(Lưu ý: Từ Redis 6, I/O Network được đẩy sang Multi-threads, nhưng phần Execute Command vẫn là Single-thread).*

---

## 2. Memory Allocation (`jemalloc`)

Redis mặc định dùng `jemalloc` (thay vì `glibc`) trên Linux.
* **Mục đích:** Giảm thiểu "Memory Fragmentation" (phân mảnh bộ nhớ).
* Khi dữ liệu liên tục bị xóa/sửa, RAM sẽ bị rỗ. `jemalloc` gom lại cực kỳ hiệu quả, giúp tiết kiệm RAM đáng kể cho in-memory DB.

---

## 3. Data Structures & Internals

| Data Type | External Use | Internal Implementation (Encoding) | Tại sao? |
|---|---|---|---|
| **String** | Cache, Counter | **SDS (Simple Dynamic String)** | Nhớ độ dài O(1), Binary-safe, tránh buffer overflow. |
| **Hash** | User profile | Small: **listpack**<br/>Large: **Hash Table** | Cỡ nhỏ dùng listpack ép thành 1 khối memory cho gọn. Cỡ to lấy O(1) để lookup. |
| **List** | Message queue | **quicklist** | Kết hợp giữa Doubly-Linked-List và listpack (chống phân mảnh). |
| **Set** | Unique tags | Small: **intset**<br/>Large: **Hash Table** | Mảng số nguyên cực kỳ tiết kiệm bộ nhớ. |
| **Sorted Set** | Leaderboard | **Skip List + Hash Table** | Skip List tìm kiếm O(log N) hiệu quả hơn Balanced Tree, Hash table lấy score O(1). |

---

## 4. Kiến Trúc Triển Khai (Deployment Modes)

```mermaid
graph TB
    subgraph "1. Standalone"
        S1["Master Node"]
    end

    subgraph "2. Primary-Replica"
        M1["Master"] --> R1["Replica 1"]
        M1 --> R2["Replica 2"]
    end

    subgraph "3. Sentinel (High Availability)"
        SM["Master"] --> SR1["Replica"]
        SEN1["Sentinel 1"] -.-> SM
        SEN2["Sentinel 2"] -.-> SM
        SEN3["Sentinel 3"] -.-> SM
    end

    subgraph "4. Redis Cluster (Sharding)"
        C_M1["Master A<br/>Slots 0-5460"]
        C_M2["Master B<br/>Slots 5461-10922"]
        C_M3["Master C<br/>Slots 10923-16383"]
    end
```

---

## Mapping → NestJS

| Đặc điểm Redis | NestJS Implementation |
|---|---|
| **In-memory cache** | `@nestjs/cache-manager` + `cache-manager-redis-store` |
| **Single-threaded atomic** | Dùng LUA Script hoặc Redis Transaction (MULTI/EXEC) để khóa resource |
| **IO Multiplexing** | Node.js dùng libuv (epoll) → Kiến trúc hoàn toàn tương đồng Redis! |
| **Distributed Lock** | `redlock` npm package (để NestJS scaling ko đụng nhau) |
