# Database Advanced (SQL & Postgres) – Mid-Level Developer Interview Reference

---

## 🗄️ Section 1: Relational Database Architecture & Internals

---

### 1. ACID vs BASE: Choosing the Right Data Model

**ACID (Atomicity, Consistency, Isolation, Durability):**
-   **Definition:** Traditional SQL databases (PostgreSQL, MySQL). Every transaction is guaranteed to be "All or Nothing".
-   **Pros:** Data integrity is absolute. Essential for financial and inventory systems.

**BASE (Basically Available, Soft state, Eventual consistency):**
-   **Definition:** NoSQL databases (Cassandra, MongoDB). Prioritizes availability and speed over immediate consistency.
-   **Pros:** Massive horizontal scalability.
-   **Mid-Level Nuance:** You must choose based on the user's needs. If they can tolerate a "slightly out of date" follower count for 2 seconds, use BASE. If they need to know their exact account balance, use ACID.

---

### 2. Deep Dive: PostgreSQL Indexing Strategy

**B-Tree (Default):** Used for equality (`=`) and range (`<`, `>`, `BETWEEN`) queries. Best for 90% of use cases.
**GIN (Generalized Inverted Index):** Essential for **Full-Text Search** and **JSONB** indexing. It stores "pointers" to where specific keys or words exist.
**GiST (Generalized Search Tree):** Best for **Geospatial data** (Lat/Long) and "Nearest Neighbor" searches.
**BRIN (Block Range Index):** Used for **Massive Timeseries tables** (billions of rows) where data is naturally sorted by time. It uses 1% of the space of a B-Tree but is only efficient for range scans.

---

### 3. MVCC (Multi-Version Concurrency Control)

**Definition:** Modern databases like Postgres don't "lock" a row when you read it. Instead, they provide a **Snapshot** of the database at the time the transaction started.

**Internals (The "Shadow" Row):**
1.  When you `UPDATE` a row, Postgres doesn't overwrite it. 
2.  It marks the old row as "invisible" and creates a **New Row Version** for the update.
3.  **READERS** keep reading the old version while the **WRITER** works on the new one.
4.  **VACUUM:** Eventually, Postgres "cleans up" the old invisible rows to free up space. This is why "Auto-vacuum" performance is a common interview topic.

---

### 4. Transaction Isolation Levels (The "Read" Anomalies)

1.  **Read Committed (Default):** You only see data that has been committed by other transactions. Prevents "Dirty Reads".
2.  **Repeatable Read:** If you read a row twice in one transaction, the value is guaranteed to be the same, even if another transaction committed an update in between. Prevents "Non-repeatable reads".
3.  **Serializable:** The strictest level. It simulates that transactions run one-after-another. Prevents "Phantom Reads".
    -   *Impact:* Highest safety but lowest performance due to locking/conflicts.

---

### 5. Optimistic vs Pessimistic Locking

-   **Pessimistic Locking (`SELECT FOR UPDATE`):** The record is "locked" the moment you read it. No one else can even read/update it until you finish.
    -   *Use Case:* High-contention scenarios (e.g., "Last 1 ticket available" for a concert).
-   **Optimistic Locking (**Version column):** You don't lock anything. You just read the record + a `version` number. When saving: `UPDATE ... WHERE version = 5`. If it fails, someone else updated it first, so you retry.
    -   *Use Case:* Low-contention scenarios (e.g., a "Profile Update" where two users rarely edit at the exact same millisecond).

---

### 6. Sharding vs Partitioning (Horizontal vs Vertical Scaling)

**Table Partitioning (Vertical):** Splitting one massive table into smaller sub-tables based on a key (e.g., `orders_2023`, `orders_2024`).
-   *Benefit:* Faster queries as the database only scans the relevant partition.

**Sharding (Horizontal):** Splitting the entire database across **multiple physical servers**. 
-   *Benefit:* Unlimited scale.
-   *Cons:* Extremely complex. Joins across shards are slow/impossible. Usually handled by "NewSQL" databases like CockroachDB or Citus (for Postgres).

---

### 7. JSONB vs Structured Columns (The Postgres Hybrid)

**JSONB Advantage:** 
1.  **Binary Format:** Faster to parse than plain JSON.
2.  **Indexing:** You can build a GIN index on specific keys inside the JSONB.
3.  **Evolution:** Great for "Profile/Settings" where fields change weekly.

**When NOT to use JSONB:**
-   If you have a fixed schema, structured columns are **always faster**, use 50% less space, and provide strict data types (`INTEGER`, `BOOLEAN`).

---

### 8. EXPLAIN ANALYZE: How to debug a 20-second query?

**Workflow:**
1.  Run `EXPLAIN ANALYZE <your query>`.
2.  **Look for "Seq Scan":** This means the database is scanning every single row (No Index used!).
3.  **Look for "Index Scan":** Good! It's using an index.
4.  **Check "Cost" vs "Actual Time":** Sometimes the query planner makes a mistake and picks the wrong index.
5.  **Fix:** Add missing indexes, rewrite the query to avoid complex sub-queries, or use a "Common Table Expression" (CTE) for clarity.

---

### 9. Master-Slave Replication vs Multi-Master

-   **Master-Slave (Read Replicas):** 1 "Write" database and 3 "Read Only" databases.
    -   *Benefit:* Offloads massive read traffic (e.g., `analytics`) from the main database.
-   **Multi-Master:** Every server can accept writes.
    -   *Cons:* Conflict resolution is nightmares (e.g., two users editing the same row on different servers at the same time). Use only if you have massive global scale.

---

### 10. Database Connection Pooling (PgBouncer)

**Problem:** Postgres creates a new **operating system process** for every client connection. This consumes ~10MB of RAM per connection. If you have 10,000 clients, you will crash the server.

**The Solution:** Use a **Connection Pooler** (like PgBouncer).
-   Clients connect to the pooler.
-   The pooler "recycles" a small number of real database connections (e.g., 50 real connections for 5,000 clients).
-   Essential for **Serverless (Lambda)** functions.

---

### 11. Deadlocks: What are they and how to avoid them?

**Definition:** A state where Transaction A holds a lock that Transaction B needs, AND Transaction B holds a lock that Transaction A needs.
-   **Impact:** Both transactions wait forever until the database kills one (the "Deadlock Victim").

**Prevention:**
1.  **Lock Order:** Always update tables in the **Same Order** (e.g., always `Users` then `Accounts`).
2.  **Short Transactions:** Keep transactions small to hold locks for as little time as possible.
3.  **Limit SELECT FOR UPDATE:** Only use pessimistic locks when strictly necessary.

---

### 12. De-normalization: When is it okay?

**Definition:** Normalization (1NF, 2NF, 3NF) reduces redundancy. De-normalization intentionally adds redundancy for **Performance**.

**Example:**
Storing a `total_orders` count in the `Users` table instead of running `COUNT(*)` across 1 million orders every time you view a profile.
-   **Benefit:** O(1) read instead of O(N) scan.
-   **Cost:** You must now use a **Database Trigger** or an **Application hook** to keep the count in sync.

---

### 13. Foreign Keys: Handling Deletes

-   **CASCADE:** Delete the user → Delete all their orders. (Dangerous but clean).
-   **SET NULL:** Delete the user → Set `user_id` in orders to `NULL`. (Good for keeping history).
-   **RESTRICT:** Prevent deletion of the user if they have orders. (Safest for audit-critical data).

---

### 14. View vs Materialized View

-   **View:** A "Saved Query". Every time you call it, the database runs the query. (Always fresh data, but no performance gain).
-   **Materialized View:** A "Physical Table" that stores the query result.
    -   **Benefit:** Extremely fast for complex reports (joins/aggregations).
    -   **Cons:** Data gets stale. You must manually call `REFRESH MATERIALIZED VIEW`.

---

### 15. Postgres Full-Text Search (tsvector)

**Why not `LIKE %search%`?** 
-   `LIKE` with a leading `%` cannot use a standard index. It's O(N).
-   Postgres FTS creates a **Search Index** using Lexemes (root words). Searching for "Swimming" will also match "Swim". It is O(log N) and infinitely faster than `LIKE`.

---

### 16. Replication Lag: Read-after-Write Consistency

**Problem:** You write to the Master, then immediate `GET` from the Read Replica. The replica is 100ms behind, so the user sees "User Not Found".

**Solution:**
1.  **Stick to Master:** Route "Profile" reads to the Master for 1 minute after an update.
2.  **Versioning:** Pass a version number to the client. If the replica's version is lower, fallback to Master or wait.

---

### 17. Database Migrations: Zero Downtime

**Problem:** Adding a column to a 100M row table can "Lock" the table for 10 minutes, crashing your app.

**Strategy:**
1.  **Add Column (Nullable):** Doesn't lock the table.
2.  **Backfill:** Update old rows in small batches (e.g., 1000 at a time).
3.  **Deploy Code:** Change the app to write to BOTH old and new fields.
4.  **Drop Old:** Finally, drop the old column once everything is stable.

---

### 18. CTE (Common Table Expressions) vs Subqueries

**CTE (`WITH my_data AS (...)`):** 
-   **Pros:** Highly readable; allows recursion; easier to debug.
-   **Cons:** In older Postgres versions (<12), CTEs were an optimization fence (the DB couldn't optimize across the boundaries).

**Subquery:** 
-   Often faster in older DBs but unreadable when nested 3 levels deep.

---

### 19. SQL Window Functions

**Definition:** Perform a calculation across a set of rows related to the current row without grouping them.
-   **`ROW_NUMBER() OVER(PARTITION BY category)`**: Assigns a unique number to rows within a category. 
-   **Use case:** "Get the top 3 most recent orders for EVERY user in one query".

---

### 20. Database Backup: Logical vs Physical

-   **Logical (`pg_dump`):** Generates a text file of SQL commands (`INSERT INTO ...`). Best for small DBs and versioning.
-   **Physical (WAL / Binary):** Copies the actual data files. 
    -   *Benefit:* Extremely fast for multi-terabyte DBs. Required for **Point-in-Time Recovery (PITR)**.

---

### 21. Change Data Capture (CDC)

**Definition:** Using tools like **Debezium** to listen to the Database's "Write Ahead Log" (WAL). 
-   Whenever a row is inserted/updated, a message is sent to **Kafka**.
-   **Use case:** Keeping Elasticsearch in sync with SQL without your application code having to talk to ES.

---

### 22. Scaling Reads: Cache-Aside Pattern

**Workflow:**
1.  App checks Redis. 
2.  If Miss, Query SQL.
3.  Write result to Redis for future.
4.  **Invalidation:** On `UPDATE`, you **MUST** delete the Redis key to prevent stale data.

---

### 23. Upsert: `ON CONFLICT`

Instead of `IF (exists) { UPDATE } ELSE { INSERT }` (which has a race condition), use:
`INSERT INTO users ... ON CONFLICT (email) DO UPDATE SET last_login = now();`
-   **Benefit:** Atomic and handles race conditions at the database level.

---

### 24. Database "Hot Spots"

**Problem:** If you use a sequential ID (`1, 2, 3...`) as a primary key, all new writes land on the same "Disk Page".
-   **Mid-Level Solution:** Use **UUIDs** or **Snowflake IDs** to randomize the write locations, spreading the load across the entire disk and preventing I/O bottlenecks.

---

### 25. High Availability (HA) vs Disaster Recovery (DR)

-   **HA:** Making sure the DB keeps running if a server fails (using **Failover** to a stand-by replica).
-   **DR:** Making sure you can get data back if the whole Data Center burns down (using **Cross-Region Backups**).

---

*Expansion Complete*
