# Module 3: Distributed Systems & Reliability

This module explores the complexities of building systems that span multiple nodes, the consensus protocols that keep them in sync, and the resilience patterns required to survive partial failures in hostile network environments.

---

## The Fundamentals of Distribution

### 31. Explain the CAP theorem in depth and how modern databases handle the 'P' (Partition Tolerance).
**Context**: The fundamental law of distributed data systems. It states you can only have two of the three: Consistency (C), Availability (A), and Partition Tolerance (P).

**Deep Dive**:
- **The Reality of 'P'**: You *cannot choose not to have* Partition Tolerance. Networks will drop packets, switches will fail. Therefore, in a distributed system, you are actually choosing between **CP** (Consistency when a partition occurs) or **AP** (Availability when a partition occurs).
    - *CP Example (MongoDB, HBase, FoundationDB)*: If a network split occurs between Node 1 and Node 2, the system refuses reads/writes (loses availability) to guarantee no client reads stale data.
    - *AP Example (Cassandra, DynamoDB, Riak)*: If a network split occurs, both sides of the split continue to accept reads/writes. They remain available, but risk returning stale, divergent data (sacrificing consistency).
- **Modern Nuance**: The PACELC theorem extends CAP. Even in the absence of a partition (E), a system trades off Latency (L) and Consistency (C).

### 32. What is a consensus algorithm, and how does Raft work in practice?
**Context**: Achieving agreement on a single truth among distributed, unreliable nodes is the hardest problem in distributed systems.

**Deep Dive**:
- **Why it's needed**: If clients send concurrent writes to different nodes ($100 withdrawal sent to Node A, $50 withdrawal to Node B), how does the cluster agree on the actual balance without a single master?
- **The Raft Protocol**: Designed to be understandable. It operates in three core phases:
    1. *Leader Election*: Nodes start as "Followers" and hold a randomized election timeout. If they don't hear a heartbeat from a Leader, they become a "Candidate" and request votes. The first to get a majority (quorum, e.g., 3 out of 5) becomes Leader.
    2. *Log Replication*: The Leader accepts all client writes and appends them to its local log. It then broadcasts `AppendEntries` RPCs to all Followers.
    3. *Commitment*: The Leader does *not* acknowledge success to the client yet. Once a majority of Followers report they have safely written to their logs, the Leader "commits" the entry and tells the client "Success".
- **Real-World Applicability**: Raft powers internal state for Kubernetes (via Etcd) and HashiCorp Consul.

### 33. What is a Vector Clock, and how does it resolve conflicts?
**Context**: In AP (Highly Available) databases like Cassandra, two writes to the exact same key can happen concurrently on different nodes. Who wins?

**Deep Dive**:
- **Naïve approach**: Last-Write-Wins (LWW) based on server timestamps. Due to clock drift (NTP isn't perfect), this is extremely dangerous and can silently overwrite valid data.
- **Vector Clocks (Logical Time)**: Instead of absolute time, systems track the *causality* of events using an array of counters.
    - *Format: `[NodeA:2, NodeB:1, NodeC:0]`*
    - If Node A updates a record, it increments its counter: `[NodeA:3...]`.
    - If Node B receives that update and adds to it, it increments: `[NodeA:3, NodeB:2]`.
- **Conflict detection**: If a client reads and sees two distinct vector clocks like `[A:3, B:1]` and `[A:2, B:2]`, neither clock explicitly "happened before" the other. This is a concurrent conflict. The database throws the problem back to the client application to resolve (e.g., merging the two shopping carts together).

### 34. What are CRDTs (Conflict-free Replicated Data Types) and when would you use them over strong consensus?
**Context**: Managing conflicts at the application layer via Vector Clocks is painful. Consensus protocols like Raft are too slow for real-time edge applications.

**Deep Dive**:
- **The Concept**: CRDTs are mathematical data structures (counters, sets, graphs) that can be replicated across a network and updated independently without any coordination (no locks, no Raft). They guarantee that once all nodes sync their updates, they will mathematically converge to the exact same state, regardless of the order updates happen.
- **Use Cases**: Real-time collaborative editing (Google Docs, Figma), distributed chat typing indicators, or globally distributed shopping carts.
- **Trade-offs**: They consume large amounts of metadata memory to track tombstones (deletions) and are notoriously difficult to design securely compared to traditional DB columns.

### 35. What is the Saga Pattern (Choreography vs Orchestration)?
*(Covered conceptually in scaling, but crucial for distributed reliability).*
**Context**: A business workflow hits 4 microservices. If Service #3 crashes, the whole workflow is compromised.

**Deep Dive**:
- **Choreography (Event-Driven)**: Service A completes its task, fires an event `A_Completed`. Service B listens, does its task, fires `B_Completed`. 
    - *Pros*: Completely decoupled. No central brain.
    - *Cons*: Impossible to trace the workflow visually. A cyclical event loop will crash the system silently.
- **Orchestration (Command-Driven)**: A central "Saga Execution Coordinator" (SEC) like AWS Step Functions or temporal.io is created. The SEC tells Service A to start. It waits for the reply. It tells Service B to start.
    - *Pros*: Complete visibility into where a transaction failed. Centralized retry and rollback logic.
    - *Cons*: The orchestrator becomes a coupling point and a potential bottleneck.
- **Compensating Actions**: In both models, if Service B fails, the system must trigger a specific undo function to reverse Service A's work (e.g., refund the stripe charge).

---

## Resilience Patterns

### 36. What is the Circuit Breaker pattern, and how does it prevent cascading failures?
**Context**: If Service A calls Service B, and Service B is experiencing a 30-second database lock, Service A will exhaust all its HTTP worker threads just waiting for B to respond. Soon, Service A crashes.

**Deep Dive**:
- A Circuit Breaker (like Resilience4j or Envoy mesh proxies) sits between the caller and the receiver.
    1. **Closed**: Normal state. It counts failures over a sliding window (e.g., last 100 calls).
    2. **Open**: If failure rate exceeds a threshold (e.g., 50%), the breaker "opens". It stops sending requests to B entirely, instantly returning an error or a fallback response to the client. This saves Service A's threads and gives Service B time to recover to drop load.
    3. **Half-Open**: After a timeout (e.g., 30s), it allows exactly *one* request through. If it succeeds, the breaker closes. If it fails, it opens again.
- **Real-world applicability**: Mandatory for any synchronous service-to-service HTTP/gRPC call.

### 37. What is the Bulkhead isolation pattern?
**Context**: Derived from ship engineering, where watertight compartments ensure a leak in one section doesn't flood the whole ship.

**Deep Dive**:
- **Implementation**: In software, it means severely isolating resources.
    1. *Thread Pool Bulkheads*: Do not use a single global thread pool for handling all incoming HTTP requests. If the API endpoint for "Heavy Image Processing" backs up, it will consume the entire thread pool, preventing users from logging in. Give the image API its own restricted thread pool (e.g., max 10 threads).
    2. *Hardware Bulkheads*: Separate critical infrastructural components onto distinct clusters. Never put the high-burst analytics DB on the same physical SAN as the payment processing DB.

### 38. How do you implement Distributed Tracing context propagation?
**Context**: Observability is part of reliability. When a user checkout fails across 6 microservices, logs are useless without a thread tying them together.

**Deep Dive**:
- **The Concept**: Tracing involves an overarching *Trace* (the entire request lifecycle) composed of *Spans* (individual service operations).
- **Context Propagation**: The API Gateway generates a unique `Trace-ID` (often using W3C Trace Context standard). It injects this ID into the HTTP headers (e.g., `traceparent`). Service A extracts it, creates a Span for its DB query, logs the result with the `Trace-ID`, and injects both the `Trace-ID` and its `Span-ID` (as the parent) into the outbound HTTP call to Service B.
- **Trade-offs**: Requires modifying internal HTTP clients/interceptors across every single codebase. Must utilize sampling (only tracing 1% of successful traffic) to prevent tracing overhead from crashing the network.

### 39. What is Chaos Engineering and the "Blast Radius"?
**Context**: You cannot guarantee fault tolerance by writing code; you must test it by breaking the system violently in production.

**Deep Dive**:
- **Chaos Engineering**: Pioneered by Netflix (Chaos Monkey). It is the disciplined approach of injecting artificial failures (terminating EC2 instances, injecting 500ms network latency via proxy filters, dropping packets) to observe how automated recovery systems react.
- **Blast Radius**: The core rule of Chaos Engineering. You start with the smallest possible blast radius (e.g., injecting latency into exactly 1 endpoint for exactly 1 test user account). As confidence grows, the radius expands. The goal is to discover hidden dependencies (e.g., finding out the Frontend completely crashes and white-screens if the tertiary "Recommend Product" API returns a 500).

### 40. Graceful Degradation vs Fallbacks
**Context**: The system must provide value even when heavily impaired.

**Deep Dive**:
- **Fallbacks (Technical)**: A localized code path executed via a Circuit Breaker. "If Redis Cache is unreachable, execute actual DB SQL query."
- **Graceful Degradation (Product)**: A holistic system behavior. "If the massive Recommendation Engine ML cluster is down, don't crash the homepage. Fall back to returning static 'Top 10 Global Sellers' cached from JSON." The user experiences a *degraded* but functional experience rather than a 500 error page.
