# Redis Basics & Caching – Mid-Level Developer Interview Reference

---

## ⚡ Section 1: Redis Architecture & Speed

---

### 1. Why is Redis so fast? (Internal mechanics)

**Definition:** Redis is an in-memory, key-value data store.

**The Speed Secret:**
1.  **In-Memory:** No slow disk I/O. Accessing RAM is 100,000x faster than accessing an SSD.
2.  **Single-Threaded Event Loop:** (Mid-level nuance). By using a single thread, Redis avoids "Context Switching" and "Locking" (no Mutexes/Semaphores needed). It uses non-blocking I/O (epoll) just like Node.js.
3.  **Highly Optimized Data Structures:** Redis's data structures (like skip-lists for Sorted Sets) are designed for speed, not space efficiency.

---

### 2. Core Data Structures: When to use which?

1.  **Strings:** Basic key-value (e.g., `session:123`).
2.  **Lists:** Ordered collections (e.g., `recent_notifications`). Good for LRU lists.
3.  **Sets:** Unordered unique collections (e.g., `unique_user_ips`).
4.  **Hashes:** Perfect for Objects (e.g., `user:101 { name: "Alice", age: 30 }`). Much more memory-efficient than storing JSON strings for multi-field objects.
5.  **Sorted Sets (ZSET):** Sets with a "Score". (e.g., **Leaderboards**, **Rate Limiting**).

---

### 3. Persistence: RDB vs AOF

-   **RDB (Redis Database):** Takes a snapshot of the whole dataset every X minutes.
    -   *Pros:* Fast restarts for large datasets; compact file size.
    -   *Cons:* You lose data since the last snapshot if the server crashes.
-   **AOF (Append Only File):** Records every write operation as it happens.
    -   *Pros:* Extremely durable (almost zero data loss).
    -   *Cons:* Large file size; slower to restart as Redis must "replay" every command.

---

### 4. Caching Pattern: Cache Aside (Lazy Loading)

**Workflow:**
1.  App checks Redis. 
2.  If Miss, Query SQL.
3.  Write result to Redis for future.
4.  **Wait! What about updates?** On SQL Update, you MUST **Delete** the Redis key. Do NOT try to update it (avoiding race conditions).

---

### 5. What is the "Cache Stampede" (Dog-piling)?

**Problem:** A very popular key (e.g., `homepage_data`) expires. Suddenly, 1,000 concurrent requests see a cache-miss and ALL try to query the Database at the exact same millisecond, potentially crashing the DB.

**The Solution:**
1.  **External Lock (Redlock):** Only the first request gets to query the DB; others wait.
2.  **Jitter (Random TTL):** Don't expire all 1 million keys at exactly 1 hour. Use `1hour + rand(0, 10mins)`.

---

### 6. Redis Eviction Policies (Memory Full!)

When Redis is full, what does it delete to make room?
1.  **NoEviction:** Standard. It just returns an error for new writes.
2.  **AllKeys-LRU:** Deletes the "Least Recently Used" key across everything.
3.  **Volatile-LRU:** Only deletes the LRU keys that **Have an Expiry (TTL)** set. (Safest).
4.  **LFU (Least Frequently Used):** Deletes keys that are "rarely touched" even if they were used recently.

---

### 7. Pub/Sub vs Redis Streams

-   **Pub/Sub:** "Fire and Forget". If a consumer is offline when a message is sent, they **Miss it forever**. (Good for real-time notifications).
-   **Streams:** Persistence built-in. Messages stay in Redis. Consumers can "Catch up" on old messages. (Good for transactional message queues).

---

### 8. Transactions: MULTI/EXEC vs Lua Scripts

-   **MULTI/EXEC:** Commands are queued and executed as a block. 
    -   *Problem:* No "Rollback" if one command fails. Subsequent commands still run!
-   **Lua Scripts:** (Preferred). A Lua script is sent to the Redis server and is **guaranteed** to be atomic. No other command can run while the script is executing.

---

### 9. Why are Lua scripts "Atomic" and "Blocking"?

**Internal:** Since Redis is single-threaded, if a Lua script takes 5 seconds, the whole Redis server is "Frozen" for those 5 seconds. 
-   **Mid-level tip:** Keep Lua scripts extremely fast (O(1) or small O(N)). Never do heavy loops in Lua inside Redis.

---

### 10. The "Big Keys" Problem

**Problem:** A single key (like a Hash with 1 million fields) is so large that fetching it eats up the entire network bandwidth or locks the single thread for too long.

**Solution:** 
1.  Use `redis-cli --bigkeys` to find them.
2.  Split the big hash into smaller "Sharded Hashes" (e.g., `user:1:meta`, `user:1:stats`).

---

### 11. Redis Replication: Master-Slave vs Sentinel

**Master-Slave:** Provides **Read Replicas**. High read performance but no automatic failover. 
**Redis Sentinel:** Provides **High Availability**. It monitors your Master. If the Master dies, Sentinel automatically promotes a Slave to be the new Master and tells your application to switch IPs.

---

### 12. Redis Cluster: Data Sharding

**Definition:** Scaling Redis horizontally across multiple servers.
-   **Hash Slots:** Redis Cluster has **16,384 hash slots**. 
-   Each key is assigned to a slot: `CRC16(key) % 16384`.
-   Servers own "ranges" of slots (e.g., Server A owns 0-5000).
-   **Benefit:** You can have a 1TB Redis cluster by splitting data across 10 servers.

---

### 13. HyperLogLog: Memory-Efficient Counting

**Scenario:** You want to count "Unique Daily Users" (UV) for a page with 100 million visitors.
-   **Set approach:** Storing 100M IDs in a Set takes **Gigabytes**.
-   **HyperLogLog approach:** Uses only **12KB** of memory.
-   **Trade-off:** It's an "Approximate" count within 0.81% error. (Perfect for analytics).

---

### 14. Geospatial Indexes (GEOADD / GEORADIUS)

**Use Case:** "Find users within 5km of me".
-   Redis stores lat/long as **Sorted Sets** internally using **Geohashing**. 
-   It's extremely efficient for proximity searches in real-time apps like Uber or Tinder.

---

### 15. Key Invalidation: "Broadcasting" on SET

**Problem:** Your Node.js app also has a "Small Local Cache". When the global Redis key changes, how do you tell all 50 Node instances to clear their local cache?
-   **Solution:** Use **Pub/Sub** to "Broadcast" an invalidation message whenever a key is updated in Redis.

---

### 16. What is "Pipelining"?

**Problem:** Sending 100 commands one-by-one results in 100 "Network Round-trips". (100ms total).
**Solution:** Pipelining sends all 100 commands in a **single packet**. Redis processes them all and sends all 100 results back in one packet. (1ms total).
-   **Mid-level tip:** This is the #1 performance boost for bulk inserts.

---

### 17. Redis as a "Distributed Lock" (Redlock)

**Problem:** Two servers trying to process a "Refund" at the exact same time.
**Redlock Algorithm:** 
1.  Try to set a key `lock:user:1` with an expiry.
2.  If successful, you "own" the lock.
3.  Process the data.
4.  Delete the key.
-   **Why the expiry?** If your server crashes half-way, the lock will automatically clear after 10 seconds, preventing a "Deadlock".

---

### 18. Memory Fragmentation

**Definition:** Redis allocates memory but when it deletes keys, the OS doesn't always take the memory back immediately. Redis sees "1GB used" but the OS says "Redis is using 2GB".
-   **Solution:** Use `CONFIG SET activedefrag yes`. Redis will internally reorganize its memory pointers to free up blocks for the OS.

---

### 19. Naming Best Practices (Namespaces)

Always use a consistent schema with colons: `<app>:<environment>:<module>:<id>`.
Example: `pulse:prod:user:123:session`.
-   **Benefit:** Allows easy filtering using `SCAN` commands and much better visualization in tools like RedisInsight.

---

### 20. When to use Redis vs Memcached?

-   **Memcached:** Better for simple string caching with high thread concurrency. (Legacy).
-   **Redis:** Better for **Complex Data Structures**, **Persistence**, and **Advanced Messaging**. (Modern Standard).

---

*Expansion Complete*
