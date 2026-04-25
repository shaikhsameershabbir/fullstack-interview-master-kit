## 16) Databases (MongoDB + Mongoose, SQL basics, transactions & indexing)

### Definition (technical)
A **database** is a persistent data store with query capabilities and consistency guarantees. **SQL databases** model data relationally (tables, schema, joins) with ACID transactions, while **document databases** like MongoDB store semi-structured documents with different modeling and query trade-offs. **Indexes** accelerate lookups; **transactions** provide atomicity across multiple operations.

In backend work, your database is usually the real “core” of the system.

Teacher truth:
> Most performance problems are database problems wearing a Node.js costume.

This topic teaches the mental models you need to reason about data:
- SQL vs Mongo trade-offs
- indexing (why queries get fast/slow)
- transactions (how to keep data correct)

---

### Basic intuition (simple)
A database is your system of record. It must be:
- correct
- consistent enough for your business rules
- fast enough under load

Two broad families:
- **SQL (relational)**: tables, schema, joins, strong transactional model
- **NoSQL (MongoDB)**: documents, flexible schema, different query and scaling trade-offs

Teacher checkpoint:
> Pick a database based on your access patterns and consistency needs — not based on hype.

---

### Internal working (engine level)
#### Indexing: why “where” becomes cheap
An index is a data structure that makes lookups fast.

Without an index, the DB often has to:
- scan lots of rows/documents
- filter them
- then return matches

With an index, it can:
- jump directly to likely matches
- read far fewer records

Trade-offs:
- indexes speed reads
- but slow writes (index maintenance)
- and consume memory/disk

Teacher rule:
> Every index is a bet: “we will query like this often enough to pay the write/memory cost.”

#### Transactions: keeping multi-step updates correct
Transactions bundle operations into “all or nothing.”

Why you need them:
- you update A then B
- if you crash after updating A but before B, you corrupt invariants

SQL: transactions are fundamental.  
Mongo: transactions exist (especially in replica sets) but have overhead and design implications.

Teacher checkpoint:
> Transactions protect correctness. Indexes protect speed. You need both — but for different reasons.

---

### Edge cases (real backend pitfalls)
- “Flexible schema” becomes “data chaos” without validation.
- Bad/missing indexes cause:
  - slow queries
  - high CPU on the DB
  - timeouts in Node
- N+1 query patterns (too many small queries) destroy performance.

---

### Interview questions
- What is an index? When does it help and when does it hurt?
- What is a transaction and why does it exist?
- SQL vs Mongo: what trade-offs do you consider for a backend service?
- What’s the N+1 problem and how do you avoid it?

---

### Real-world usage (Node.js)
- Measure query performance:
  - explain plans
  - slow query logs
  - p95/p99 latencies
- Use connection pooling (don’t open a new connection per request).
- Validate inputs at boundaries (DTOs/schema validation).
- Put timeouts and retries around DB calls thoughtfully (avoid retry storms).

---

### Practice (do these in Node)
1) Create a “users” query that filters by email; add an index and compare query time.
2) Write a two-step update (debit one account, credit another). Explain why it must be transactional.
3) Simulate N+1 by querying inside a loop; refactor to a single query/batched approach.

---

### Connections to other topics
- **19 Performance**: DB is often the bottleneck; Node can only be as fast as its dependencies.
- **20 Scaling**: caching reduces DB load; replicas/sharding change architecture.
- **23 System design**: data modeling choices dominate architecture.
