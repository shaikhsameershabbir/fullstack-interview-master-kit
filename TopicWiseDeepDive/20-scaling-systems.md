## 20) Scaling Systems (load balancing, stateless apps, caching/Redis)

### Definition (technical)
**Scaling systems** is increasing capacity and reliability under higher load by distributing traffic across instances (load balancing), minimizing per-instance state (stateless design), and reducing dependency pressure/latency via techniques like caching (e.g., Redis), replication, and asynchronous processing.

Scaling is not “add more servers.” Scaling is:

> Handle more traffic while keeping latency and reliability acceptable.

Teacher truth:
- You can’t scale a stateful app easily.
- You can’t scale a slow database by only adding Node instances.

So we’ll learn the core ideas: load balancing, statelessness, caching.

---

### Basic intuition (simple)
- Scaling means handling more traffic with acceptable latency and reliability.
- The simplest scaling strategy is **horizontal scaling**: run more instances behind a load balancer.

### Internal working (engine level)
#### Load balancing
Distributes traffic across instances (round-robin, least connections, etc.).
Implications:
- you can’t rely on “this request will always hit the same server” unless sticky sessions are used.

#### Stateless design
If your app stores state in memory (sessions, caches), scaling becomes hard.
Stateless apps store state in shared systems:
- DB
- Redis
- object storage
- message queues

#### Caching (Redis)
Caching improves latency and reduces load on DB/services.
Common patterns:
- read-through / cache-aside
- write-through
- TTL-based caching

### Edge cases
- Cache invalidation is hard (stale data bugs).
- Hot keys and cache stampedes under load.
- Over-caching can hide correctness issues; always define consistency requirements.

### Interview questions
- What does it mean for an app to be stateless?
- How does a load balancer help?
- What caching strategy would you use for X and why?

### Real-world usage (Node.js)
- Use Redis for sessions, rate limiting, caching, job queues.
- Design endpoints to be idempotent where possible to handle retries.

### Practice (do these in Node)
1) Design session storage for an Express app that runs 5 instances. Where do sessions live and why?
2) Implement a cache-aside pattern for a “get user by id” endpoint (pseudo).
3) Explain what happens if your cache goes down (fallback + load spike).

### Connections to other topics
- **Authentication**: session stores vs JWT change statefulness.
- **Message queues**: for async workflows and decoupling.
- **Performance**: caching is often the highest ROI optimization.
