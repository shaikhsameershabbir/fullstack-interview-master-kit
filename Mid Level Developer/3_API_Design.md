# API Design & Best Practices – Mid-Level Developer Interview Reference

---

## 🌐 Section 1: RESTful Architecture & Advanced Patterns

---

### 1. What are the core REST API Best Practices (Architectural view)?

**Definition:** REST (Representational State Transfer) is an architectural style, not a strict protocol. It relies on a stateless, client-server communication model.

**High-Level Best Practices:**
1.  **Statelessness:** The server should not store any client context between requests. Each request must contain all the information necessary to understand and process the request (e.g., Auth tokens in headers).
2.  **Resource-Oriented Design:** Use nouns (`/customers`, `/invoices`) rather than verbs (`/getInvoices`). The "verb" is provided by the HTTP method (`GET`, `POST`, `PUT`, `PATCH`, `DELETE`).
3.  **Logical Hierarchy & Nesting:** `/users/102/orders/5` clearly represents "Order #5 belonging to User #102". Limit nesting to 2-3 levels max to avoid overly complex URLs.
4.  **HATEOAS (Hypermedia as the Engine of Application State):** A truly RESTful API provides links in the response body that guide the client on what they can do next (e.g., `"_links": { "payment": "/orders/123/pay" }`).

---

### 2. Idempotency and the "Idempotency Key" Pattern

**Definition:** An operation is **idempotent** if it can be performed multiple times without changing the result beyond the initial application.

**The "Double Charge" Problem:** 
In payment systems, a network timeout might occur after the server processes a payment but before the client receives the `200 OK`. If the client retries, they might get charged twice.

**The Solution (Idempotency Key):**
1.  The client generates a unique UUID (the Idempotency Key) and sends it in the header: `X-Idempotency-Key: uuid-v4`.
2.  The server checks its cache (Redis) for this key.
3.  If found, it returns the **cached response** instead of re-processing.
4.  If not found, it processes the request and stores the result in Redis for a set TTL (e.g., 24 hours).

---

### 3. API Pagination: Offset vs Cursor (Deep Dive)

**Offset Pagination (`LIMIT 10 OFFSET 50`):**
-   **Internal:** The database must scan through the first 50 rows and discard them before returning the next 10.
-   **Pros:** Easy to implement; allows "jumping" to Page 10.
-   **Cons:** Extremely slow on large tables (O(N) complexity). Inconsistent results if rows are deleted/added between page loads (the "missing item" or "duplicate item" bug).

**Cursor Pagination (`WHERE created_at < 'last_seen_timestamp' LIMIT 10`):**
-   **Internal:** The database uses an index to jump directly to the last seen item and fetches the next 10.
-   **Pros:** Constant time (O(1)) performance regardless of dataset size. Highly consistent for infinite scroll.
-   **Cons:** Complex to implement; cannot "jump" to a specific page.

---

### 4. API Versioning Strategies: URL vs Headers

**1. URL Versioning (`/v1/users`):**
-   **Pros:** Most visible; easy to cache at the CDN/Proxy level; easy for developers to test.
-   **Cons:** Pollutes URI space; requires updating all client URLs on a major change.

**2. Header / Media Type Versioning (`Accept: version=2.0`):**
-   **Pros:** "Clean" URLs; allows "Evolution" where different parts of the API evolve at different rates.
-   **Cons:** Harder to test; breaks browser-native caching (as the URL is the same but the content changes).

**Mid-Level Consensus:** **URL Versioning** is the industry standard for public APIs due to its simplicity and cache-friendliness.

---

### 5. The BFF (Backend for Frontend) Pattern

**Definition:** Instead of one "Generic API" for all clients, you build a specific backend service for each client type (Mobile, Web, IoT).

**Architectural Benefits:**
1.  **Payload Optimization:** Mobile might only need 5 fields, while Web needs 50. BFF handles the filtering.
2.  **Request Aggregation:** A "Product Details" page might need data from 4 microservices (Price, Stock, Reviews, Description). Instead of the Mobile app making 4 network calls, it makes 1 call to the BFF, which aggregates the data internally.
3.  **Third-Party Shielding:** If you switch from Stripe to PayPal, you only update the BFF, not the mobile app (which requires an App Store review).

---

### 6. Designing for Resilience: The Circuit Breaker Pattern

**Problem:** If a downstream service (e.g., the Shipping API) is slow or down, your main API will hang while waiting for the timeout, eventually exhausting all its threads/memory.

**The Solution:**
1.  **Closed State:** Requests flow normally.
2.  **Open State:** After X consecutive failures, the circuit "trips" (opens). All subsequent requests fail **immediately** with a `503 Service Unavailable` without waiting for a timeout.
3.  **Half-Open State:** After a "sleep" period, the circuit allows a few "test" requests. If they succeed, the circuit closes again.

---

### 7. API Documentation as Code: OpenAPI / Swagger

**Definition:** OpenAPI is a language-agnostic specification for describing REST APIs. 

**Why it matters:**
-   **Source of Truth:** It provides a contract that both Frontend and Backend teams agree upon.
-   **Auto-Generation:** You can generate interactive UI (Swagger UI) where developers can "Try it out" live.
-   **Client SDK Generation:** You can auto-generate TypeScript or Swift models from the spec, ensuring types are always in sync.

---

### 8. Webhooks vs WebSockets vs Long Polling

-   **Polling:** Client asks every 5 seconds: "Is it ready?". (High overhead, simple).
-   **WebSockets:** Full-duplex persistent connection. (Best for Chat, Games, real-time dashboards).
-   **Webhooks:** **Server calls the Client**. When a "Charge Succeeded" event happens in Stripe, Stripe makes a `POST` request to YOUR server. (Best for asynchronous events like payment processing).

---

### 9. Rate Limiting: Token Bucket vs Fixed Window

-   **Fixed Window:** "100 requests per minute". (Risk: a user can send 100 requests in the last second of Minute 1 and 100 in the first second of Minute 2, effectively doubling the rate).
-   **Token Bucket:** The server gives the user a "bucket" that refills with 1 token every second. Each request costs 1 token. If the bucket is empty, the request is rejected. (Handles "Bursts" of traffic smoothly).

---

### 10. API Monitoring: SLIs and SLOs

**SLI (Service Level Indicator):** A specific metric (e.g., "P99 Latency of `/login`").
**SLO (Service Level Objective):** The target goal (e.g., "99.9% of `/login` requests must return within 200ms").

**Why P99?** 
Average (mean) latency is misleading because 10 very slow users (the "long tail") might be having a terrible experience while the average looks fine. P99 focuses on the slowest 1% of users.

---

### 11. API Evolution: "Expando-Only" Strategy

**Problem:** How do you update an API without bumping the version from `/v1` to `/v2`?

**Strategy:**
1.  **Never Delete/Rename Fields:** Always add new fields.
2.  **Make New Fields Optional:** So old versions of the app don't crash when they don't see them.
3.  **Graceful Deprecation:** Add a `Warning` header in the response to let developers know a field is deprecated and will be removed in 6 months.

---

### 12. Handling "Large File Uploads" (Direct vs Multi-part)?

**Mid-level way (S3 Pre-signed URLs):**
-   Client asks API: "I want to upload a file".
-   API asks S3: "Give me a temporary upload link".
-   API returns the link to the client.
-   Client uploads **directly** to S3.
-   **Benefit:** Zero load on your Node.js server. Your server only handles small JSON messages. 

---

### 13. Async Job Status Pattern

**Problem:** A client requests an action that takes 30 seconds (e.g., "Generate PDF Report").

**Solution:**
1.  Server returns `202 Accepted` immediately.
2.  Server provides a `job_id` and a status URL: `/jobs/456`.
3.  The client "polls" the status URL until it sees `status: "completed"`.
4.  The final result link is then provided in the job response.

---

### 14. JWT Security: Scopes vs Claims

**Definitions:**
-   **Claims:** Pieces of information assertions about the subject (e.g., `email`, `sub`, `exp`). They are "Who you are".
-   **Scopes:** A set of permissions associated with the access token (e.g., `read:users`, `write:orders`). They are "What you can do".

**Security Best Practice:** A mid-level developer should ensure that the API checks **Both**. Just because you have a valid token doesn't mean you have the `scope` to perform a destructive action.

---

### 15. Problem Details for HTTP APIs (RFC 7807)

**Definition:** A standard format for error responses. instead of `{ "error": "Not Found" }`, you use a standardized JSON object.

**Example:**
```json
{
  "type": "https://example.com/probs/out-of-stock",
  "title": "Out of Stock",
  "status": 400,
  "detail": "The Item 'Blue Shirt' is currently unavailable.",
  "instance": "/orders/123/items/4"
}
```
**Benefit:** Allows the client to programmatically handle errors using the `type` URI while still being human-readable.

---

### 16. API Orchestration vs API Choreography

-   **Orchestration (Centralized):** A single "Orchestrator" service calls Service A, then Service B, then Service C. If one fails, the orchestrator handles the rollback. (Easier to monitor, but single point of failure).
-   **Choreography (Decentralized):** Services communicate via a **Message Queue**. Service A finishes and publishes an event. Service B hears the event and starts. (Highly scalable, but harder to visualize the whole process).

---

### 17. Conditional Requests: Etags and `If-None-Match`

**Definition:**
1.  Server sends a `ETag` (hash of the content) header.
2.  Client stores this tag.
3.  Next time, Client sends `If-None-Match: "etag_value"`.
4.  If content hasn't changed, Server returns **304 Not Modified** (zero body size).

**Benefit:** Massive bandwidth savings and slightly faster response times as the server doesn't have to re-encode the JSON.

---

### 18. Designing "Soft Delete" via API

**Strategy:**
Instead of `DELETE /users/1` removing the row, it sets `deleted_at = timestamp`.
-   **API Design:** Subsequent `GET /users/1` should return **404 Not Found** (or a specific "User was deleted" message).
-   **Admin API:** Might allow `GET /users/1?include_deleted=true`.
-   **GDPR Tip:** Even with soft-delete, you must provide a way for "Hard Delete" (Right to be Forgotten).

---

### 19. Handling "Conflict" (409) vs "Precondition Failed" (412)

-   **409 Conflict:** Used when the request conflicts with the current state of the server (e.g., trying to register an email that already exists).
-   **412 Precondition Failed:** Used with **Optimistic Locking**. If you send `If-Match: "old_etag"` and the server's etag has changed (someone else updated it), the server returns 412.

---

### 20. JSON Patch (RFC 6902) vs JSON Merge Patch (RFC 7386)

-   **Merge Patch:** `{ "name": "New Name" }`. Simple, just merges the keys. **Problem:** Can't easily set a value to `null` if the library ignores nulls.
-   **JSON Patch:** `[{ "op": "replace", "path": "/name", "value": "New Name" }]`. 
    -   **Benefit:** Precise. Can delete, move, add, or replace specific array elements without sending the whole array.

---

### 21. API Analytics: Metrics beyond Latency

A mid-level developer should track:
1.  **Error Rate per Client:** Is one specific user hitting 401s repeatedly? (Potential attack or bug in their code).
2.  **Usage Quotas:** How close is the user to their monthly 10k request limit?
3.  **TTR (Time to Resolution):** How long does a background job typically stay in "pending"?
4.  **Endpoint Popularity:** Which versions/endpoints are no longer used (safe to deprecate)?

---

### 22. Designing "Public" vs "Internal" APIs

-   **Internal APIs:** Can be more coupled, use faster protocols (gRPC), and have less strict versioning. Often live inside the VPC.
-   **Public APIs:** Must have extensive documentation, strict versioning, strong rate-limiting, and high security. Always behind an API Gateway with WAF (Web Application Firewall).

---

### 23. Contract Testing (Pact)

**Problem:** How do you know that changing a field in the User Service won't break the Order Service?

**Solution:** **Contract Testing**. 
1.  The Consumer (Order Service) defines a "contract" of what it expects.
2.  The Provider (User Service) runs this contract against itself before every deployment.
3.  If the change breaks the contract, the build fails. (Much faster than full E2E integration tests).

---

### 24. GraphQL: Schema Federation

**Definition:** Instead of one massive GraphQL server, you split the schema into "subgraphs" owned by different teams (e.g., `Users Subgraph`, `Products Subgraph`).
-   A **Gateway** (or Router) merges these subgraphs into one unified API for the client.
-   **Benefit:** Teams can deploy their subgraphs independently without affecting others.

---

### 25. "Payload Compression" Trade-offs

**Gzip / Brotli:**
-   **Pros:** Reduces network transfer time significantly.
-   **Cons:** Consumes **CPU** on the server to compress and on the client to decompress.
-   **Mid-Level Tip:** For very small JSON payloads (< 1KB), compression might actually make the request **slower** due to the CPU overhead. Only enable it for payloads above a certain threshold (e.g., 1KB).

---

### 26. Designing for "High Availability": Multi-Region APIs

**Definition:** Running your API in two or more geographic regions (e.g., `us-east-1` and `eu-west-1`) simultaneously.

**Key Challenges:**
1.  **Global Load Balancing:** Using Route 53 (DNS) or Global Accelerator to route users to the nearest healthy region.
2.  **Data Replication:** How to sync the database? (e.g., DynamoDB Global Tables or Aurora Global Database).
3.  **Conflict Resolution:** What if two users edit the same row in different regions at the same time? (Last Write Wins vs. CRDTs).

---

### 27. Security: OWASP Top 10 for APIs (BOLA & BFLA)

1.  **BOLA (Broken Object Level Authorization):** Attacher changes `/api/orders/123` to `/api/orders/456`. If the API only checks "Are you logged in?" but not "Do you own order 456?", that's a BOLA attack.
2.  **BFLA (Broken Function Level Authorization):** An regular user tries to hit `POST /api/admin/users`.
-   **Mid-Level Fix:** Always check permissions **and** ownership at the controller level for every single request.

---

### 28. "Graceful Degradation" Pattern

**Scenario:** Your "Recommendation Engine" is down.
-   **Bad API:** Returns `500 Internal Server Error` for the whole home page.
-   **Mid-Level API:** Catches the error from the recommendation service and returns a **default list** (e.g., "Top Selling Items") instead. The user doesn't even know something is broken.

---

### 29. API Style Guides & Spectral (Linting)

**Problem:** Team A uses `camelCase` and Team B uses `snake_case`. Team C uses `404` for missing users and Team D uses `204`.
**Solution:** **Spectral**. It's a "Linter" for your OpenAPI/Swagger files. It automatically fails the build if your API spec doesn't follow the company's rules (e.g., "All endpoints must have a 401 response defined").

---

### 30. Performance: Request Hedging

**Problem:** 99% of requests are fast (100ms), but 1% are slow (2 seconds) due to random network jitter or GC pauses.
**Solution:** If a request hasn't returned in 150ms (the P95 latency), the client (or BFF) sends a **second identical request**. Whichever returns first is used.
-   *Impact:* Dramatically reduces the P99 latency at the cost of slightly more server load.

---

### 31. Security: JWT Revocation Strategies

**Problem:** JWTs are stateless. If a user logs out or is banned, their token is still valid until it expires.
**Strategies:**
1.  **Blacklisting (Redis):** Store the `jti` (unique ID) of revoked tokens in Redis until they expire. Check Redis on every request.
2.  **Short TTLs:** Make tokens live for only 5 minutes. Use a **Refresh Token** (stored in the DB) to get new ones. To ban a user, just delete their Refresh Token.

---

### 32. CQRS (Command Query Responsibility Segregation)

**Definition:** Separating the "Write" logic from the "Read" logic at the API level.
-   **Command API:** `POST /orders` - Optimized for high-throughput writes.
-   **Query API:** `GET /order-history` - Optimized for complex joins and cached for high-speed reads. 
-   **Benefit:** You can scale them independently (e.g., 10 Read servers for 1 Write server).

---

### 33. Outbox Pattern for Reliable Events

**Problem:** You save an order to the DB and then try to send a "Order Created" event to Kafka. The Kafka call fails. Now the DB is out of sync with your events.
**Solution:** 
1.  Save the Order **and** an "Outbox Message" in the same SQL Transaction.
2.  A separate background worker polls the Outbox table and sends messages to Kafka.
3.  Ensures "Exactly Once" or "At Least Once" delivery of events.

---

### 34. Designing "HATEOAS": The Navigation Logic

**Definition:** The API response includes URIs for related actions.
**Example:**
```json
{
  "order_id": 123,
  "status": "pending",
  "_links": {
    "self": "/orders/123",
    "cancel": "/orders/123/cancel",
    "pay": "/orders/123/payments"
  }
}
```
**Benefit:** The frontend doesn't need to "hardcode" URLs. It just follows the links provided by the server. This makes the API much easier to evolve.

---

### 35. Security: CSRF (Cross-Site Request Forgery) in APIs

**Does a REST API need CSRF protection?**
-   If you use **Cookies** for authentication, **YES**.
-   If you use **Authorization Headers** (JWT), **NO**. 
-   **Why?** Browsers automatically send cookies with every request, but they **cannot** automatically attach a custom `Authorization` header.

---

### 36. Testing: API Fuzzing

**Definition:** Automatically sending "Garbage" or "Edge Case" data to your API to find crashes or security holes.
-   Example: Sending a 10MB string to a "Name" field, or sending emojis to an "Age" field. 
-   Tools like **AFL** or **Restler** help automate this.

---

### 37. API Gateway vs Load Balancer

-   **Load Balancer (Layer 4/7):** Just distributes traffic across servers.
-   **API Gateway (Layer 7):** Does **Authentication**, **Rate Limiting**, **Request Transformation**, and **Metric Collection**. It is the entry point for your entire microservice ecosystem.

---

### 38. "Database per Service" in API Design

**Problem:** If 10 services share one DB, one slow service can crash them all.
**Rule:** Every microservice should own its own database that NO other service can talk to. Communication must happen via **APIs** or **Message Queues**. This is the only way to achieve true architectural decoupling.

---

### 39. Managing "API Quotas" for Third Parties

**Mechanism:** 
-   Assign a `tier` to each API Key (Free, Pro, Enterprise).
-   Use a Redis `INCR` command with a TTL of 1 month.
-   Example: `SET quota:key_abc:2023_10 0 EX 31days`.
-   If `INCR` exceeds the tier's limit, return `429 Too Many Requests`.

---

### 40. "Backend for Frontend" (BFF) for Micro-frontends

If you have a complex dashboard with a "Shipping Widget" and a "Reviews Widget" (Micro-frontends), each widget should have its own mini-BFF. 
-   **Benefit:** The Shipping team can change their API without ever coordinating with the Reviews team or the main dashboard team.

---

*Expansion Complete (40 Questions)*
