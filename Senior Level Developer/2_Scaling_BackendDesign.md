# Module 2: Scaling Systems & Backend System Design

This module focuses on the engineering challenges of hyper-scale systems (1M+ users/requests) and the meticulous design required for critical backend services like payments, real-time architectures, and connection management.

---

## Scaling Systems & Bottlenecks

### 21. How do you design a system to handle 1 million Requests Per Second (RPS)?
**Context**: Handling 1M RPS is rarely a single-machine code challenge; it's a distributed infrastructure and networking challenge. Standard setups collapse under connection limits long before CPU maxes out.

**Deep Dive**:
- **Connection & Port Exhaustion**: A load balancer or NAT gateway only has 65,535 ports available to connect to backend targets. At 1M RPS, you will hit SNAT port exhaustion.
    *Mitigation*: Use Direct Server Return (DSR), where the load balancer forwards the packet directly, and the backend server responds directly to the client, bypassing the LB on the return trip.
- **Protocol Optimization**: HTTP/1.1 requires a new TCP connection (or multiplexing limits) per request. At 1M RPS, the TCP handshakes alone will kill the network.
    *Mitigation*: Use HTTP/2 or HTTP/3 (QUIC/UDP) for connection multiplexing. Internally, move all service-to-service communication to gRPC using long-lived TCP connection pools.
- **Data Layer Saturation**: No single RDBMS can handle 1M write RPS.
    *Mitigation*: Aggressive batching (Kafka -> micro-batches -> DB) and heavy sharding strategies. 

**Trade-offs**: Micro-optimization at this scale significantly increases code complexity. Moving from synchronous HTTP to asynchronous event streams makes debugging errors exponentially harder.

### 22. How do you handle database connection pooling at massive scale?
**Context**: A PostgreSQL database assigns a new OS process (using ~10MB RAM) for every connection. If 1,000 Lambda functions scale up, they can instantly open 10,000 connections, crashing the database with "Out of Memory" errors.

**Deep Dive**:
- **Application-side Pooling (HikariCP, generic-pool)**: Good for traditional long-lived server systems (EC2/Containers), where max connections are strictly bounded locally.
- **Proxy-side Pooling (PgBouncer, AWS RDS Proxy)**: Mandatory for Serverless or extreme high-burst architectures. A proxy sits between the app and the DB. The app connects to the proxy (creating thousands of idle connections cheaply), and the proxy multiplexes those requests onto a tiny, fixed pool (e.g., 200) of actual DB connections.
- **Transaction vs Session mechanisms**: When using PgBouncer, "Transaction Pooling" must be used so connections are returned to the pool immediately after the `COMMIT`, not when the client disconnects.

**Real-World Applicability**: Essential for any rapidly auto-scaling architecture connecting to a traditional RDBMS.

### 23. Design a scalable Rate Limiter system.
**Context**: APIs must be protected from abuse (DDoS, scraper bots, or runaway internal scripts). Rate Limiters restrict requests per client per time window.

**Deep Dive**:
- **Algorithms**:
    1. *Token Bucket*: A bucket holds a max number of tokens. Tokens are added at a fixed rate. A request consumes a token. Good for allowing short bursts.
    2. *Fixed Window*: Counts requests in fixed blocks (e.g., 12:00:00 to 12:01:00).
        *Cons*: Subject to traffic spikes at the edges of the window.
    3. *Sliding Window Log*: Stores a timestamp for *every* request in Redis. Highly accurate but consumes massive memory.
- **Distributed Implementation**: Implementing a Rate Limiter across multiple API Gateway nodes requires centralized state.
    *Solution*: Use **Redis**. Use Lua scripts to evaluate the exact Token/Window logic atomically on the Redis server, preventing race conditions when two concurrent requests try to decrement the bucket simultaneously.

**Trade-offs**: A centralized Redis rate limiter introduces a single point of failure and latency. For ultra-high scale, use a local, in-memory cache per node combined with eventual synchronization to a cluster (like the "Leaky Bucket" mixed with gossip protocol).

### 24. How do you handle cache invalidation at scale (Cache Stampede & Thundering Herd)?
**Context**: "There are only two hard things in Computer Science: cache invalidation and naming things."

**Deep Dive**:
- **The Problem (Cache Stampede)**: A highly accessed key (e.g., "Trending Topics") expires in Redis. Instantly, 50,000 requests hit the cache, see a "miss," and all 50,000 query the primary Database simultaneously to rebuild the cache. The DB instantly dies.
- **Mitigation Strategies**:
    1. *Probabilistic Early Expiration (XFetch)*: A server artificially considers a cache key "expired" a few seconds *before* the actual TTL expires, based on a random probability. Only one server will "win" this lottery and asynchronously update the cache while the others still use the slightly stale data.
    2. *Mutex Locking*: When a cache miss occurs, the server tries to acquire a Redis lock. If it wins, it queries the DB. If it fails, it waits 50ms and checks the cache again.
    3. *Background Refresh*: The application never reads from the DB directly on a cache miss. A background cron-job constantly queries the DB every 10 seconds to warm the cache.

---

## Critical Backend Systems

### 25. How would you design a payment processing system with strict consistency?
**Context**: Payments require absolute precision. A lost record means stolen money; a duplicate record means double billing.

**Deep Dive**:
- **Double-Entry Bookkeeping**: Never use an `UPDATE accounts SET balance = balance + 50`. It leaves no history. Always use `INSERT INTO ledger (account_id, amount, side)` where `side` is Debit or Credit. The balance is a `SUM` of the ledger stream.
- **State Machines**: A payment shouldn't just be "Success" or "Failed." It goes through rigorous states: `INITIALIZED` -> `AUTH_PENDING` -> `CAPTURED` -> `SETTLED` -> `RECONCILED`.
- **ACID over CAP**: This is the one place where relational databases (PostgreSQL) and strong ACID guarantees trump NoSQL and Eventual Consistency.

**Real-world applicability**: Even if your system is microservices-based, payment ledgers often remain as tightly coupled, transactional monoliths to guarantee exact precision.

### 26. How do you ensure idempotency in distributed payments?
**Context**: Network failures lead to UI retries. If a user clicks "Pay" and their internet drops, the app will retry. You cannot charge them twice.

**Deep Dive**:
- **Idempotency Key Protocol**: The client generates a UUID (v4) and sends it as a header `X-Idempotency-Key` along with the payload hash.
- **Database Level Enforcement**:
    1. Create an `idempotency_keys` table: `(key, status, response_body, created_at)`.
    2. When a request hits, begin a database transaction.
    3. Try to `INSERT` the key. If it violates a `UNIQUE` constraint, catch the error.
        - If the status is `COMPLETED`, return the saved `response_body` (HTTP 200).
        - If the status is `IN_PROGRESS`, return `HTTP 409 Conflict` (client should wait and retry).
    4. If it succeeds, process the payment with the gateway.
    5. Save the result to `response_body`, mark `COMPLETED`, and commit the transaction.

### 27. Two-Phase Commits (2PC) vs. Sagas for distributed transactions.
**Context**: You need to deduct inventory in Service A and charge a card in Service B. If the card charge fails, you must release the inventory.

**Deep Dive**:
- **Two-Phase Commit (2PC)**: A coordinator service locks the database rows in both Service A and Service B. It asks "Are you ready?" (Prepare Phase). If both say yes, it says "Commit."
    *Cons*: It blocks resources. If the coordinator dies midway, entire databases can remain locked indefinitely. Not suitable for modern microservices.
- **Saga Pattern (Choreography/Orchestration)**: Local transactions combined with compensating actions.
    - *Forward*: Decrease Inventory (local commit) -> Emit Event -> Charge Card (local commit).
    - *Compensation*: If card charge fails, emit `PaymentFailedEvent` -> Inventory Service listens, and increments inventory back to original (local commit).

**Trade-offs**: Sagas introduce eventual consistency. There is a small window where inventory is deducted but money is not yet collected. It also requires building complex "rollback" (compensating) methods for every action.

### 28. How do you design a high-throughput webhook dispatching system?
**Context**: You process a payment and need to notify external clients via webhooks. Clients might be offline, slow, or returning 500 errors.

**Deep Dive**:
- **Decoupling Delivery**: Never send webhooks synchronously during the API request. Persist the event to an "Outbox Table" or publish to Kafka.
- **Worker Pools & Queues**: Use a dedicated background worker pool reading from a queue (RabbitMQ/SQS).
- **Exponential Backoff**: If the client responds with `503`, requeue the message with a delay (`delay = base * 2^attempt`). Add random "jitter" to avoid hammering the client when they come back online.
- **Security Check**: Only dispatch webhooks with HMAC signatures in the header, calculated using an API Secret, so the client can verify the payload wasn't tampered with.

### 29. What are the strategies for handling massive WebSocket concurrent connections?
**Context**: You are building a live chat app (like WhatsApp or Discord) needing 1 million concurrent, idle-but-open connections.

**Deep Dive**:
- **The C10M Problem**: The OS bottleneck is keeping ports and file descriptors open.
    *Mitigation*: Tune kernel parameters (`fs.file-max`, `net.ipv4.ip_local_port_range`). Use non-blocking I/O frameworks (Node.js, Go, Netty in Java) that utilize `epoll`/`kqueue` to manage thousands of sockets on a single thread.
- **Stateless Routers**: A WebSocket connection is inherently stateful (pinned to one server). When "User A" sends a message to "User B", how does Server 1 know User B is connected to Server 80?
    *Mitigation*: Use a pub/sub backplane (Redis Pub/Sub or Kafka). All servers subscribe to a channel. Server 1 publishes the message to Redis; Server 80 sees it, checks its local HashMap of active connections, finds User B, and pushes the socket message.

### 30. How do you design a Search System with near-real-time indexing?
**Context**: Relational databases fail miserably at full-text search across millions of records. An inverted index is needed, but syncing it is hard.

**Deep Dive**:
- **The Tech**: Use Elasticsearch or OpenSearch. They map terms to a list of IDs where that term occurs (Inverted Index).
- **The Synchronization Challenge (CDC)**: Dual-writing (writing to SQL, then writing to Elastic from your app) is an anti-pattern prone to race conditions and out-of-sync states on network failure.
    *Mitigation*: Use the **Transactional Outbox Pattern** or Change Data Capture (CDC via Debezium). Debezium listens to the PostgreSQL Write-Ahead Log (WAL), instantly streams the insert/update to Kafka, which seamlessly consumes and indexes it in Elasticsearch. This guarantees eventual consistency.
