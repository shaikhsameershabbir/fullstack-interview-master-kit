## 23) System Design (API design at scale, rate limiting, DB scaling)

### Definition (technical)
**System design** is the process of defining a system’s components, interfaces, data flows, and operational characteristics to meet requirements (latency, throughput, availability, consistency, security, cost), including failure handling, scaling strategies, and evolvable API/data models.

System design is where you stop thinking about “a server” and start thinking about “a system.”

Teacher truth:
> At scale, failures are normal. Retries are normal. Uneven traffic is normal.  
> Good design is about making the system behave well under those realities.

We’ll focus on practical fundamentals:
- API design (contracts, pagination, idempotency)
- rate limiting (protect the system)
- database scaling (what actually works first)

---

### Basic intuition (simple)
- System design is making trade-offs: latency vs cost vs consistency vs complexity.
- “At scale” means planning for failures, retries, and uneven traffic.

### Internal working (engine level)
#### API design at scale
Key principles:
- clear contracts, versioning strategy
- idempotency for retries
- pagination, filtering, and rate limits
Bad APIs create operational pain: huge payloads, ambiguous errors, breaking changes.

#### Rate limiting
Protects your service from abuse and accidental overload.
Common algorithms:
- token bucket
- leaky bucket
- fixed window / sliding window

Usually implemented with shared storage (Redis) in distributed setups.

#### DB scaling (high level)
Approaches:
- indexing and query optimization (first)
- read replicas
- caching
- sharding (hard)

### Edge cases
- Rate limits without good error messaging frustrate legitimate users.
- DB scaling without data model changes often fails; schema and access patterns are everything.

### Interview questions
- Design an API for X with pagination and filtering.
- How would you implement rate limiting?
- How do you scale reads vs writes in a database-backed system?

### Real-world usage (Node.js)
- Node services often sit behind gateways; rate limiting and auth are usually centralized.
- Observability (logs/metrics/traces) is part of “system design”, not an afterthought.

### Practice (design exercises)
1) Design an API endpoint for listing orders:
   - pagination
   - filtering by status/date
   - sorting
   Explain how clients can safely retry.
2) Design rate limiting for login endpoint vs public read endpoint. Why different?
3) Propose a DB scaling plan for “users table is slow” starting from indexing.

### Connections to other topics
- **Scaling systems**: load balancing, statelessness, caching are system design basics.
- **Auth**: security and identity are core constraints.
- **Message queues**: async workflows and decoupling.
