# Module 2: API Architecture & GraphQL Data Fetching

This module transitions from the runtime environment into how backend systems expose data to clients. Senior engineers must understand the trade-offs between REST and GraphQL, the necessity of pagination, and the N+1 database problem.

---

## 4️⃣ Advanced API Design

### 11. What defines a truly RESTful Architecture (HATEOAS)?
**Deep Dive**: 
- Most APIs are "HTTP APIs" (CRUD over HTTP), not actual REST. True REST (Representational State Transfer) adheres to the Richardson Maturity Model Level 3: **HATEOAS (Hypermedia as the Engine of Application State)**.
- **Implementation**: When you hit `GET /users/1`, the JSON response doesn't just contain `{ id: 1, name: "Bob" }`. It contains a `_links` object explicitly telling the UI what actions it can take next (e.g., `"update_url": "/users/1/update"`, `"delete_url": "/users/1/delete"`). The client doesn't need to hardcode API routes; it dynamically navigates the API based on the links provided.

### 12. Offset Pagination vs Cursor Pagination at hyper-scale.
**Deep Dive**: 
- **Offset/Limit (Standard)**: `SELECT * FROM users OFFSET 10000 LIMIT 50`.
    - *The Problem*: The database still has to scan and discard the first 10,000 rows. At page 10,000, the query takes seconds. Additionally, if a new item is inserted while the user clicks "Page 2", items shift, causing duplicate entries in the UI.
- **Cursor Pagination (Hyper-scale)**: `SELECT * FROM users WHERE id > "cursor_100" LIMIT 50`.
    - *The Fix*: The DB uses the index on `id` to instantly jump to the requested row. Scanning is O(Limit) instead of O(Offset + Limit). Prevents duplicate items in real-time feeds (like Twitter/Instagram).
    - *Cons*: You cannot jump directly to page 50. You must click "Next" sequentially.

### 13. Idempotency Keys in API Design.
**Deep Dive**: 
- An operation is idempotent if executing it 100 times has the same systemic effect as executing it 1 time (e.g., `DELETE /users/1`). 
- `POST` requests (like creating a payment) are inherently non-idempotent. If the mobile app times out waiting for the server, it retries, creating two payments.
- **Solution**: The client generates a UUID `X-Idempotency-Key` header. The server stores this key and the API response in an external cache (Redis) for 24 hours. On a retry, the server sees the key, skips processing, and instantly returns the cached HTTP response.

---

## 5️⃣ GraphQL Optimization

### 14. GraphQL vs REST: The Ultimate Trade-off.
**Deep Dive**: 
- **REST Pros**: Trivial to cache using HTTP mechanics (CDNs/Varnish). Clear architectural boundaries. No chance of a client writing an infinitely deep nested query that crashes the server.
- **REST Cons**: Over-fetching (downloading 50KB of JSON when you only need a username) and Under-fetching (requiring 5 separate HTTP calls to gather user data, their posts, and comments).
- **GraphQL Pros**: The client dictates exactly what it wants in a single POST request. Self-documenting schema. Massive velocity increase for React frontend engineers.
- **GraphQL Cons**: 
    - You lose HTTP-level caching. Everything is a `POST /graphql`. You must cache manually at the application/Apollo layer.
    - Massive security risks if query depth is not maliciously rate-limited.

### 15. What are Resolvers and the infamous N+1 Problem?
**Deep Dive**: 
- **Resolvers**: Functions attached to fields in the GraphQL Schema that return the actual data for that field. 
- **The N+1 Problem**: 
    - A client queries `users(limit: 50) { posts { title } }`.
    - The top-level resolver hits the DB once: `SELECT * FROM users LIMIT 50`. (1 Query).
    - GraphQL then executes the `posts` resolver *individually* for every single user returned. It executes `SELECT * FROM posts WHERE user_id = X` fifty separate times. (N Queries). Total DB hits: 51.
- **The Fallout**: A simple API call executes a DDoS attack on your own database.

### 16. How do you solve N+1 in GraphQL? (DataLoader)
**Deep Dive**: 
- **DataLoader**: A micro-batching and caching library created by Meta.
- Instead of the `posts` resolver querying the DB immediately, it calls `dataLoader.load(user_id)`. DataLoader waits roughly 20ms (a single tick of the event loop) to collect all 50 `user_id`s that the resolvers requested.
- It then unleashes a highly optimized, single micro-batched query: `SELECT * FROM posts WHERE user_id IN (1...50)`. 
- Result: 51 DB queries are compressed into exactly 2 DB queries.

### 17. What is GraphQL Federation?
**Deep Dive**: 
- As an enterprise grows to 50 microservices, having 50 distinct REST APIs is confusing for the frontend.
- **Federation (Apollo)**: A centralized GraphQL Gateway exposes one massive "Supergraph". 
    - The User Microservice controls the `User` schema. The Review Microservice controls the `Review` schema.
    - A frontend client can write a query asking for a `User` and their `Reviews`. The Federation Gateway automatically splits the query, securely queries the User Microservice and the Review Microservice via subgraphs, stitches the JSON together, and returns it to the client. Highly seamless integration without monolithic code coupling.
