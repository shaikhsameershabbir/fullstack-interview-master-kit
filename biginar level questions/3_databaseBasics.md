# Database Basics – Complete Beginner Interview Reference

---

## 🗄️ Section 1: SQL Fundamentals

---

### 1. What is SQL?

**SQL (Structured Query Language)** is a **standardized language** used to manage and interact with **relational databases**. It lets you create, read, update, and delete data (CRUD), as well as define database schemas.

**SQL is used in:** PostgreSQL, MySQL, SQLite, Microsoft SQL Server, Oracle DB

```sql
-- SQL has 5 sub-languages:

-- DDL (Data Definition Language) — structure
CREATE TABLE users (id SERIAL PRIMARY KEY, name VARCHAR(100));
ALTER TABLE users ADD COLUMN email VARCHAR(255);
DROP TABLE users;
TRUNCATE TABLE users;  -- remove all rows, keep structure

-- DML (Data Manipulation Language) — data
INSERT INTO users (name, email) VALUES ('Alice', 'alice@mail.com');
SELECT * FROM users;
UPDATE users SET name = 'Bob' WHERE id = 1;
DELETE FROM users WHERE id = 1;

-- DCL (Data Control Language) — permissions
GRANT SELECT ON users TO readonly_user;
REVOKE SELECT ON users FROM readonly_user;

-- TCL (Transaction Control Language) — transactions
BEGIN;
COMMIT;
ROLLBACK;

-- DQL (Data Query Language) — querying
SELECT name, email FROM users WHERE age > 18 ORDER BY name;
```

---

### 2. What is a Primary Key?

A **primary key** is a column (or combination of columns) that **uniquely identifies each row** in a table. No two rows can have the same primary key value, and it **cannot be NULL**.

```sql
-- Single column primary key
CREATE TABLE users (
  id        SERIAL PRIMARY KEY,       -- auto-incrementing integer
  name      VARCHAR(100) NOT NULL,
  email     VARCHAR(255) UNIQUE NOT NULL
);

-- UUID as primary key (common in modern apps)
CREATE TABLE products (
  id    UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  name  VARCHAR(200) NOT NULL
);

-- Composite primary key (combination of columns)
CREATE TABLE order_items (
  order_id   INT,
  product_id INT,
  quantity   INT NOT NULL,
  PRIMARY KEY (order_id, product_id)  -- pair must be unique
);

-- Primary key properties:
-- ✅ Must be UNIQUE — no duplicates allowed
-- ✅ NOT NULL — cannot be empty
-- ✅ Immutable — should not change once set
-- ✅ Each table should have exactly one primary key
```

**Natural vs Surrogate keys:**
- **Natural key** — a real-world value (email, national ID) — risky if it changes
- **Surrogate key** — artificial ID (auto-increment, UUID) — recommended

---

### 3. What is a Foreign Key?

A **foreign key** is a column in one table that **references the primary key** of another table — it **creates a relationship** between tables and enforces **referential integrity**.

```sql
CREATE TABLE users (
  id    SERIAL PRIMARY KEY,
  name  VARCHAR(100) NOT NULL
);

CREATE TABLE orders (
  id         SERIAL PRIMARY KEY,
  user_id    INT NOT NULL,
  total      DECIMAL(10, 2),
  created_at TIMESTAMP DEFAULT NOW(),

  -- Foreign key constraint
  FOREIGN KEY (user_id) REFERENCES users(id)
    ON DELETE CASCADE   -- if user is deleted, delete their orders too
    ON UPDATE CASCADE   -- if user id changes, update orders too
);

-- What ON DELETE options mean:
-- CASCADE    → delete child rows when parent is deleted
-- SET NULL   → set foreign key to NULL when parent deleted
-- RESTRICT   → prevent deletion if child rows exist (default)
-- NO ACTION  → like RESTRICT (checked at end of transaction)
-- SET DEFAULT → set to a default value

-- Inserting related data
INSERT INTO users (name) VALUES ('Alice');   -- id = 1
INSERT INTO orders (user_id, total) VALUES (1, 99.99);  -- references user 1

-- This would FAIL — no user with id = 999
INSERT INTO orders (user_id, total) VALUES (999, 50.00);
-- ERROR: insert or update violates foreign key constraint
```

**Foreign key = referential integrity = data consistency**

---

### 4. What is Indexing?

An **index** is a **data structure** (usually a B-tree) created on one or more columns that speeds up **data retrieval** at the cost of slightly slower writes and more storage.

Think of it like a book's **index page** — instead of reading every page, you look up the term in the index.

```sql
-- Without index: database scans ALL rows (full table scan) → slow
-- With index:    database jumps directly to matching rows → fast

-- Create an index
CREATE INDEX idx_users_email ON users (email);
CREATE INDEX idx_orders_user_id ON orders (user_id);

-- Unique index (also enforces uniqueness)
CREATE UNIQUE INDEX idx_users_email_unique ON users (email);

-- Composite index — good for queries filtering on multiple columns
CREATE INDEX idx_orders_user_date ON orders (user_id, created_at);

-- Full-text search index (PostgreSQL)
CREATE INDEX idx_products_name_fts ON products USING GIN (to_tsvector('english', name));

-- Primary keys are automatically indexed!
-- Foreign keys should usually be indexed too.

-- View indexes on a table (PostgreSQL)
SELECT indexname, indexdef FROM pg_indexes WHERE tablename = 'users';

-- Remove an index
DROP INDEX idx_users_email;
```

**When to index:**
- Columns frequently used in `WHERE`, `JOIN`, `ORDER BY`, `GROUP BY`
- Foreign key columns
- Columns used for searching/filtering

**When NOT to index:**
- Small tables (full scan is fast enough)
- Columns rarely queried
- Columns with low cardinality (e.g., a boolean — only 2 values)
- Tables with very frequent writes (indexes slow down INSERT/UPDATE/DELETE)

---

### 5. What is Normalization?

**Normalization** is the process of organizing a relational database to **reduce data redundancy** and improve **data integrity** by following a series of rules called **Normal Forms (NF)**.

#### First Normal Form (1NF):
- Each column contains **atomic (indivisible) values**
- No repeating groups or arrays

```sql
-- ❌ NOT 1NF — multiple values in one column
| user_id | name  | phones                  |
|---------|-------|-------------------------|
| 1       | Alice | 9876543210, 9123456789  |

-- ✅ 1NF — one value per column
| user_id | name  | phone      |
|---------|-------|------------|
| 1       | Alice | 9876543210 |
| 1       | Alice | 9123456789 |
```

#### Second Normal Form (2NF):
- Must be in 1NF
- Every non-key column depends on the **entire primary key** (no partial dependencies)

```sql
-- ❌ NOT 2NF — product_name depends only on product_id, not full PK
-- PK is (order_id, product_id)
| order_id | product_id | product_name | quantity |
|----------|-----------|--------------|----------|

-- ✅ 2NF — separate tables
orders:   (order_id, product_id, quantity)
products: (product_id, product_name)
```

#### Third Normal Form (3NF):
- Must be in 2NF
- No **transitive dependencies** (non-key column depends on another non-key column)

```sql
-- ❌ NOT 3NF — city depends on zip_code (not on user_id)
| user_id | zip_code | city    |
|---------|----------|---------|

-- ✅ 3NF — separate zip code table
users:     (user_id, zip_code)
zip_codes: (zip_code, city)
```

**Benefits of normalization:** Less redundancy, less storage, easier updates
**Downside:** More JOINs needed — sometimes **denormalize** for read performance

---

### 6. What is a JOIN?

A **JOIN** combines rows from two or more tables based on a **related column** (usually a foreign key relationship).

```sql
-- Sample tables:
-- users:  id, name
-- orders: id, user_id, total, status

-- Basic JOIN syntax
SELECT columns
FROM table1
JOIN table2 ON table1.column = table2.column;

-- Example: Get each order with the user's name
SELECT
  u.name        AS customer_name,
  o.id          AS order_id,
  o.total,
  o.status
FROM orders o
JOIN users u ON o.user_id = u.id;

-- Multiple JOINs
SELECT
  u.name,
  o.total,
  p.name AS product_name
FROM orders o
JOIN users u    ON o.user_id = u.id
JOIN products p ON o.product_id = p.id
WHERE o.status = 'completed';
```

**Types of JOINs:**

| JOIN | Returns |
|---|---|
| `INNER JOIN` | Only matching rows in BOTH tables |
| `LEFT JOIN` | All from left + matched from right (NULL if no match) |
| `RIGHT JOIN` | All from right + matched from left (NULL if no match) |
| `FULL OUTER JOIN` | All rows from BOTH tables |
| `CROSS JOIN` | Every combination of rows (Cartesian product) |
| `SELF JOIN` | Table joined with itself |

---

### 7. Difference Between INNER JOIN and LEFT JOIN?

```sql
-- Tables:
-- users:  { id: 1, name: "Alice" }, { id: 2, name: "Bob" }, { id: 3, name: "Charlie" }
-- orders: { id: 1, user_id: 1, total: 99 }, { id: 2, user_id: 2, total: 149 }
-- (Charlie has no orders)

-- INNER JOIN — only users WHO HAVE orders
SELECT u.name, o.total
FROM users u
INNER JOIN orders o ON u.id = o.user_id;

-- Result:
-- name  | total
-- Alice | 99
-- Bob   | 149
-- (Charlie is excluded — no matching order!)

-- LEFT JOIN — ALL users, even if they have no orders
SELECT u.name, o.total
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;

-- Result:
-- name    | total
-- Alice   | 99
-- Bob     | 149
-- Charlie | NULL   ← included with NULL for missing order data

-- RIGHT JOIN — all orders, even if no matching user (rare)
SELECT u.name, o.total
FROM users u
RIGHT JOIN orders o ON u.id = o.user_id;

-- FULL OUTER JOIN — everything from both tables
SELECT u.name, o.total
FROM users u
FULL OUTER JOIN orders o ON u.id = o.user_id;

-- Find users WITHOUT any orders
SELECT u.name
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.id IS NULL;  -- Charlie (NULL means no order matched)
```

---

### 8. What is NoSQL?

**NoSQL (Not only SQL)** refers to a broad class of database management systems that **do not use the traditional relational table model**. They are designed for:
- Large-scale distributed systems
- Flexible/evolving data models
- High throughput
- Horizontal scaling

**4 Main types of NoSQL databases:**

| Type | Structure | Examples | Best for |
|---|---|---|---|
| **Document** | JSON-like documents | MongoDB, CouchDB | Content management, user profiles |
| **Key-Value** | Simple key → value | Redis, DynamoDB | Caching, sessions |
| **Column-family** | Column-oriented rows | Cassandra, HBase | Analytics, time-series |
| **Graph** | Nodes and edges | Neo4j, ArangoDB | Social networks, recommendations |

**MongoDB (Document DB) example:**
```js
// Document — flexible, JSON-like
{
  "_id": "64abc123",
  "name": "Alice",
  "email": "alice@mail.com",
  "address": {
    "city": "Mumbai",        // nested!
    "zip": "400001"
  },
  "orders": [               // arrays of data!
    { "id": 1, "total": 99 },
    { "id": 2, "total": 149 }
  ],
  "tags": ["vip", "premium"]
}
```

---

### 9. SQL vs NoSQL?

| Feature | SQL (Relational) | NoSQL (Non-relational) |
|---|---|---|
| **Schema** | Fixed, predefined | Flexible, dynamic |
| **Data model** | Tables, rows, columns | Documents, key-value, graphs |
| **Relationships** | JOINs between tables | Embedding or referencing |
| **Scaling** | Vertical (bigger server) | Horizontal (more servers) |
| **ACID** | ✅ Full ACID support | ⚠️ Varies (MongoDB has ACID for transactions) |
| **Query language** | SQL (standardized) | Database-specific APIs |
| **Performance** | Slower for huge distributed datasets | Faster for large, simple reads/writes |
| **Best for** | Complex queries, financial data, reporting | Big data, real-time, flexible schemas |
| **Examples** | PostgreSQL, MySQL, SQLite | MongoDB, Redis, Cassandra |

```
Use SQL when:             Use NoSQL when:
✅ Complex relationships   ✅ Rapidly changing schema
✅ Data integrity critical ✅ Huge scale (millions of documents)
✅ Complex reporting       ✅ Hierarchical/nested data
✅ Financial transactions  ✅ Real-time apps (chat, IoT)
✅ Standardized queries    ✅ Caching/session storage
```

---

### 10. What is MongoDB?

**MongoDB** is an open-source, document-oriented **NoSQL database**. It stores data as **BSON (Binary JSON) documents** in **collections** (like tables).

**Key Concepts:**

| SQL Term | MongoDB Equivalent |
|---|---|
| Database | Database |
| Table | Collection |
| Row | Document |
| Column | Field |
| Primary Key | `_id` field |
| JOIN | `$lookup` (aggregation) or embedding |
| Index | Index |

```js
// MongoDB shell / Mongoose commands

// --- Database operations ---
use myDatabase           // switch to database
show collections         // list collections
show dbs                 // list all databases

// --- CRUD ---
// Create
db.users.insertOne({ name: "Alice", age: 25, email: "alice@mail.com" });
db.users.insertMany([{ name: "Bob" }, { name: "Charlie" }]);

// Read
db.users.find();                           // all documents
db.users.find({ age: { $gt: 20 } });      // age > 20
db.users.findOne({ email: "alice@mail.com" });
db.users.find({}, { name: 1, email: 1, _id: 0 }); // projection

// Update
db.users.updateOne(
  { _id: ObjectId("...") },
  { $set: { age: 26 } }                   // update specific field
);
db.users.updateMany(
  { age: { $lt: 18 } },
  { $set: { isMinor: true } }
);

// Delete
db.users.deleteOne({ _id: ObjectId("...") });
db.users.deleteMany({ isActive: false });

// Count
db.users.countDocuments({ age: { $gt: 18 } });
```

---## 📝 Section 2: SQL Queries in Depth

---

### 11. What is a SELECT Statement?

`SELECT` is the most used SQL command — it retrieves data from one or more tables.

```sql
-- Basic syntax
SELECT column1, column2 FROM table_name;
SELECT * FROM users;  -- select all columns (avoid in prod — use specific columns)

-- With alias
SELECT name AS full_name, email AS contact_email FROM users;

-- Calculated columns
SELECT name, salary, salary * 1.1 AS salary_with_raise FROM employees;
SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM users;

-- Distinct — remove duplicates
SELECT DISTINCT city FROM users;
SELECT COUNT(DISTINCT city) FROM users;

-- Full SELECT order of clauses:
SELECT   columns         -- what to return
FROM     table           -- where to get data
JOIN     other_table     -- how to combine tables
WHERE    condition       -- filter rows
GROUP BY column          -- group rows for aggregation
HAVING   condition       -- filter groups
ORDER BY column ASC/DESC -- sort results
LIMIT    n               -- max number of rows
OFFSET   m;              -- skip m rows (pagination)
```

---

### 12. What is the WHERE Clause?

`WHERE` filters rows based on a condition — only rows that match are returned.

```sql
-- Basic comparisons
SELECT * FROM users WHERE age = 25;
SELECT * FROM users WHERE age != 25;
SELECT * FROM users WHERE age > 18;
SELECT * FROM users WHERE age >= 18;
SELECT * FROM users WHERE age < 65;
SELECT * FROM users WHERE salary BETWEEN 30000 AND 80000;
SELECT * FROM users WHERE name = 'Alice';

-- Multiple conditions
SELECT * FROM users WHERE age > 18 AND city = 'Mumbai';
SELECT * FROM users WHERE city = 'Mumbai' OR city = 'Delhi';
SELECT * FROM users WHERE NOT (city = 'Mumbai');

-- IN — match any value in a list
SELECT * FROM users WHERE city IN ('Mumbai', 'Delhi', 'Bangalore');
SELECT * FROM orders WHERE status NOT IN ('cancelled', 'failed');

-- LIKE — pattern matching (case-insensitive in some DBs)
SELECT * FROM users WHERE name LIKE 'A%';      -- starts with A
SELECT * FROM users WHERE name LIKE '%alice%'; -- contains alice
SELECT * FROM users WHERE email LIKE '%.com';  -- ends with .com
-- % = any sequence of characters, _ = any single character
SELECT * FROM users WHERE phone LIKE '98_______'; -- 2nd digit after 98

-- NULL checks (cannot use = for NULL!)
SELECT * FROM users WHERE phone IS NULL;
SELECT * FROM users WHERE phone IS NOT NULL;
```

---

### 13. What are Aggregate Functions?

Aggregate functions perform a **calculation on a set of rows** and return a single value.

```sql
-- COUNT — number of rows
SELECT COUNT(*)        FROM users;              -- total rows (including NULLs)
SELECT COUNT(email)    FROM users;              -- non-NULL emails only
SELECT COUNT(DISTINCT city) FROM users;         -- unique cities

-- SUM — total of a numeric column
SELECT SUM(total) FROM orders;
SELECT SUM(total) FROM orders WHERE status = 'completed';

-- AVG — average value
SELECT AVG(salary)   FROM employees;
SELECT AVG(age)      FROM users;
SELECT ROUND(AVG(age), 2) FROM users;  -- round to 2 decimal places

-- MIN and MAX
SELECT MIN(price) FROM products;
SELECT MAX(price) FROM products;
SELECT MIN(created_at), MAX(created_at) FROM orders;  -- oldest and newest

-- Combining aggregates
SELECT
  COUNT(*)    AS total_orders,
  SUM(total)  AS revenue,
  AVG(total)  AS avg_order_value,
  MIN(total)  AS smallest_order,
  MAX(total)  AS largest_order
FROM orders
WHERE status = 'completed';
```

---

### 14. What is GROUP BY and HAVING?

**`GROUP BY`** groups rows with the same value in specified columns — then you apply aggregate functions to each group.

**`HAVING`** filters **groups** (like `WHERE` but for groups — runs AFTER grouping).

```sql
-- Count orders per user
SELECT user_id, COUNT(*) AS order_count
FROM orders
GROUP BY user_id;

-- Total revenue per city
SELECT u.city, SUM(o.total) AS revenue
FROM orders o
JOIN users u ON o.user_id = u.id
GROUP BY u.city
ORDER BY revenue DESC;

-- Count users per role
SELECT role, COUNT(*) AS count
FROM users
GROUP BY role;

-- HAVING — filter groups (WHERE runs before, HAVING runs after GROUP BY)
-- Users who placed more than 3 orders
SELECT user_id, COUNT(*) AS order_count
FROM orders
GROUP BY user_id
HAVING COUNT(*) > 3;

-- Cities with total revenue > 10000
SELECT u.city, SUM(o.total) AS revenue
FROM orders o
JOIN users u ON o.user_id = u.id
GROUP BY u.city
HAVING SUM(o.total) > 10000
ORDER BY revenue DESC;

-- WHERE vs HAVING:
-- WHERE filters individual rows BEFORE grouping
-- HAVING filters groups AFTER grouping
SELECT status, COUNT(*), SUM(total)
FROM orders
WHERE created_at >= '2024-01-01'   -- filter rows first
GROUP BY status
HAVING COUNT(*) > 5;               -- then filter groups
```

---

### 15. What is ORDER BY, LIMIT, and OFFSET?

```sql
-- ORDER BY — sort results
SELECT * FROM users ORDER BY name ASC;      -- alphabetical
SELECT * FROM users ORDER BY created_at DESC; -- newest first
SELECT * FROM users ORDER BY age ASC, name ASC; -- multiple columns

-- LIMIT — restrict number of rows returned
SELECT * FROM products ORDER BY price DESC LIMIT 10;  -- top 10 most expensive

-- OFFSET — skip rows (used for pagination)
SELECT * FROM users LIMIT 10 OFFSET 0;   -- page 1 (rows 1-10)
SELECT * FROM users LIMIT 10 OFFSET 10;  -- page 2 (rows 11-20)
SELECT * FROM users LIMIT 10 OFFSET 20;  -- page 3 (rows 21-30)

-- Pagination formula
-- page 1 → LIMIT 10 OFFSET (1-1)*10 = 0
-- page 2 → LIMIT 10 OFFSET (2-1)*10 = 10
-- page N → LIMIT limit OFFSET (page-1)*limit

-- PostgreSQL shorthand
SELECT * FROM users FETCH FIRST 10 ROWS ONLY;  -- standard SQL
```

---

### 16. What are SQL Constraints?

**Constraints** are rules enforced on columns to maintain **data accuracy and integrity**.

```sql
CREATE TABLE products (
  id          SERIAL PRIMARY KEY,              -- unique + not null

  name        VARCHAR(200) NOT NULL,           -- cannot be NULL
  sku         VARCHAR(50)  UNIQUE NOT NULL,    -- must be unique

  price       DECIMAL(10,2) NOT NULL
              CHECK (price >= 0),             -- custom rule: must be >= 0

  stock       INT DEFAULT 0,                  -- default value if not provided
  category_id INT REFERENCES categories(id)  -- foreign key shorthand
              ON DELETE SET NULL,

  created_at  TIMESTAMP DEFAULT NOW()         -- auto-timestamp
);

-- Named constraints (better for error messages)
CREATE TABLE orders (
  id       SERIAL,
  total    DECIMAL(10,2),
  status   VARCHAR(20),
  user_id  INT,

  CONSTRAINT pk_orders   PRIMARY KEY (id),
  CONSTRAINT chk_total   CHECK (total >= 0),
  CONSTRAINT chk_status  CHECK (status IN ('pending', 'completed', 'cancelled')),
  CONSTRAINT fk_user     FOREIGN KEY (user_id) REFERENCES users(id)
);
```

| Constraint | Purpose |
|---|---|
| `PRIMARY KEY` | Unique + NOT NULL identifier |
| `UNIQUE` | No duplicate values |
| `NOT NULL` | Value is mandatory |
| `CHECK` | Custom validation rule |
| `DEFAULT` | Value used when none provided |
| `FOREIGN KEY` | Referential integrity between tables |

---

### 17. What is a Subquery?

A **subquery** is a SQL query **nested inside another query**. It can appear in `SELECT`, `FROM`, `WHERE`, or `HAVING` clauses.

```sql
-- Find users who have placed at least one order
SELECT name FROM users
WHERE id IN (
  SELECT DISTINCT user_id FROM orders  -- subquery
);

-- Find products more expensive than average price
SELECT name, price FROM products
WHERE price > (
  SELECT AVG(price) FROM products  -- returns a single value
);

-- Subquery in FROM (derived table)
SELECT city, avg_age
FROM (
  SELECT city, AVG(age) AS avg_age
  FROM users
  GROUP BY city
) AS city_stats
WHERE avg_age > 30;

-- Correlated subquery (references outer query — runs per row)
SELECT name, salary,
  (SELECT AVG(salary) FROM employees e2 WHERE e2.department = e1.department) AS dept_avg
FROM employees e1;

-- EXISTS — check if subquery returns any rows
SELECT name FROM users u
WHERE EXISTS (
  SELECT 1 FROM orders o WHERE o.user_id = u.id
);

-- NOT EXISTS — users with no orders
SELECT name FROM users u
WHERE NOT EXISTS (
  SELECT 1 FROM orders o WHERE o.user_id = u.id
);
```

---

### 18. What is a Transaction in SQL?

A **transaction** is a sequence of SQL operations that are **executed as a single unit** — either ALL succeed or ALL are rolled back. Maintains **ACID** properties.

```sql
-- Basic transaction
BEGIN;  -- or START TRANSACTION

  UPDATE accounts SET balance = balance - 500 WHERE id = 1;  -- debit
  UPDATE accounts SET balance = balance + 500 WHERE id = 2;  -- credit

COMMIT;  -- make changes permanent

-- If something goes wrong:
BEGIN;
  UPDATE accounts SET balance = balance - 500 WHERE id = 1;
  -- Oops! Something failed here
ROLLBACK;  -- undo all changes — balance restored!

-- Savepoints — partial rollback within a transaction
BEGIN;
  INSERT INTO users (name) VALUES ('Alice');
  SAVEPOINT after_alice;

  INSERT INTO users (name) VALUES ('Bob');
  -- Bob insert fails for some reason
  ROLLBACK TO after_alice;  -- only undo Bob's insert, keep Alice's

  INSERT INTO users (name) VALUES ('Charlie');
COMMIT;  -- Alice and Charlie are saved, Bob is not
```

---

### 19. What is ACID?

**ACID** is a set of properties that guarantee reliable database transactions:

| Property | Meaning | Example |
|---|---|---|
| **A**tomicity | All or nothing — transaction fully succeeds or fully fails | Money transfer: either both debit+credit happen, or neither |
| **C**onsistency | Database remains in a valid state before and after transaction | Balance can't go negative if constraint says >= 0 |
| **I**solation | Concurrent transactions don't interfere with each other | Two users buying last item → only one succeeds |
| **D**urability | Committed data persists even after system crashes | After COMMIT, data survives a power outage |

```sql
-- Atomicity example
BEGIN;
  UPDATE accounts SET balance = balance - 1000 WHERE id = 1;
  UPDATE accounts SET balance = balance + 1000 WHERE id = 2;
  -- If server crashes here — ROLLBACK automatically, no partial transfer
COMMIT;

-- Isolation levels (from least to most strict):
-- READ UNCOMMITTED — can read "dirty" data (uncommitted changes)
-- READ COMMITTED   — only read committed data (PostgreSQL default)
-- REPEATABLE READ  — same query returns same result within transaction
-- SERIALIZABLE     — transactions run as if sequential (strictest)

SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
BEGIN;
-- ... your queries
COMMIT;
```

---

### 20. What are SQL Data Types?

```sql
-- TEXT / STRING types
VARCHAR(n)   -- variable-length string, max n chars → 'Alice'
CHAR(n)      -- fixed-length string (padded with spaces) → 'M   '
TEXT         -- unlimited-length string (PostgreSQL)

-- NUMERIC types
INT / INTEGER     -- whole number: -2147483648 to 2147483647
BIGINT            -- large whole number
SMALLINT          -- small whole number
SERIAL            -- auto-incrementing INTEGER (PostgreSQL)
BIGSERIAL         -- auto-incrementing BIGINT
DECIMAL(p, s)     -- exact decimal: p=total digits, s=after decimal
FLOAT / DOUBLE    -- approximate decimal (floating point)
NUMERIC(10, 2)    -- e.g., 12345678.99 (currency)

-- DATE/TIME types
DATE             -- date only: '2024-03-19'
TIME             -- time only: '22:30:00'
TIMESTAMP        -- date + time: '2024-03-19 22:30:00'
TIMESTAMPTZ      -- timestamp with timezone (recommended!)
INTERVAL         -- duration: '3 hours', '2 days'

-- BOOLEAN
BOOLEAN          -- true / false / NULL

-- BINARY / JSON
BYTEA            -- binary data (PostgreSQL)
JSON             -- JSON data (stored as text)
JSONB            -- binary JSON (indexed, faster queries) ← recommended
UUID             -- universally unique identifier

-- Arrays (PostgreSQL)
TEXT[]           -- array of text → {'apple', 'banana'}
INT[]            -- array of integers → {1, 2, 3}

-- Examples
CREATE TABLE demo (
  name       VARCHAR(100),
  age        INT,
  price      DECIMAL(10, 2),
  is_active  BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  metadata   JSONB
);
```

---

## 🍃 Section 3: MongoDB in Depth

---

### 21. What are MongoDB Query Operators?

MongoDB provides powerful **query operators** for filtering, comparison, and logical conditions.

```js
// Comparison operators
User.find({ age: { $eq: 25 } });    // age == 25 (same as { age: 25 })
User.find({ age: { $ne: 25 } });    // age != 25
User.find({ age: { $gt: 18 } });    // age > 18
User.find({ age: { $gte: 18 } });   // age >= 18
User.find({ age: { $lt: 65 } });    // age < 65
User.find({ age: { $lte: 65 } });   // age <= 65
User.find({ age: { $in: [20, 25, 30] } });    // age is 20, 25, or 30
User.find({ age: { $nin: [20, 25] } });       // age NOT in list

// Logical operators
User.find({ $and: [{ age: { $gt: 18 } }, { city: "Mumbai" }] });
User.find({ $or:  [{ city: "Mumbai" }, { city: "Delhi" }] });
User.find({ $nor: [{ city: "Mumbai" }, { city: "Delhi" }] }); // neither
User.find({ age: { $not: { $gt: 18 } } }); // NOT age > 18

// Element operators
User.find({ phone: { $exists: true } });  // has phone field
User.find({ phone: { $exists: false } }); // does NOT have phone field
User.find({ age: { $type: "number" } });  // age is of type number

// String operators (with regex)
User.find({ name: /^ali/i });             // starts with "ali" (case-insensitive)
User.find({ name: { $regex: "alice", $options: "i" } });

// Array operators
User.find({ tags: "javascript" });              // array contains "javascript"
User.find({ tags: { $all: ["js", "node"] } }); // array has ALL of these
User.find({ tags: { $size: 3 } });             // array has exactly 3 elements
User.find({ "tags.0": "javascript" });          // first element is "javascript"
```

---

### 22. What is the MongoDB Aggregation Pipeline?

The **aggregation pipeline** is a powerful framework for data transformation and analysis. Data flows through a series of **stages**, each transforming the documents.

```js
// Pipeline structure: collection.aggregate([stage1, stage2, ...])

await Order.aggregate([
  // Stage 1: $match — filter documents (like WHERE)
  { $match: { status: "completed", createdAt: { $gte: new Date("2024-01-01") } } },

  // Stage 2: $group — group and aggregate (like GROUP BY)
  {
    $group: {
      _id: "$userId",                    // group by userId
      totalRevenue: { $sum: "$total" },  // sum of total field
      orderCount:   { $count: {} },      // count documents
      avgOrder:     { $avg: "$total" },  // average
      minOrder:     { $min: "$total" },
      maxOrder:     { $max: "$total" },
    }
  },

  // Stage 3: $lookup — JOIN another collection
  {
    $lookup: {
      from: "users",         // join with 'users' collection
      localField: "_id",     // from current doc
      foreignField: "_id",   // from 'users' doc
      as: "userInfo"         // store result in this field (array)
    }
  },

  // Stage 4: $unwind — flatten array from $lookup
  { $unwind: "$userInfo" },

  // Stage 5: $project — select/rename fields (like SELECT)
  {
    $project: {
      _id: 0,
      user: "$userInfo.name",
      totalRevenue: 1,
      orderCount: 1,
      avgOrder: { $round: ["$avgOrder", 2] }
    }
  },

  // Stage 6: $sort — sort results
  { $sort: { totalRevenue: -1 } },

  // Stage 7: $limit / $skip — pagination
  { $skip: 0 },
  { $limit: 10 }
]);
```

---

### 23. What is Mongoose Schema Validation?

Mongoose lets you **validate data** at the schema level before saving to MongoDB.

```js
const mongoose = require("mongoose");

const userSchema = new mongoose.Schema({
  name: {
    type: String,
    required: [true, "Name is required"],
    trim: true,
    minlength: [2, "Name must be at least 2 characters"],
    maxlength: [50, "Name cannot exceed 50 characters"],
  },
  email: {
    type: String,
    required: [true, "Email is required"],
    unique: true,
    lowercase: true,
    trim: true,
    match: [/^\S+@\S+\.\S+$/, "Please enter a valid email"],
  },
  age: {
    type: Number,
    min: [0,   "Age cannot be negative"],
    max: [150, "Age cannot exceed 150"],
  },
  role: {
    type: String,
    enum: {
      values: ["user", "admin", "moderator"],
      message: "Role must be user, admin, or moderator"
    },
    default: "user",
  },
  website: {
    type: String,
    validate: {
      validator: (v) => /^https?:\/\/.+/.test(v),
      message: "Website must be a valid URL"
    }
  },
  createdAt: { type: Date, default: Date.now },
  updatedAt: { type: Date, default: Date.now },
});

// Middleware (hooks) — run before/after operations
userSchema.pre("save", async function(next) {
  this.updatedAt = new Date();
  next();
});

userSchema.post("save", function(doc) {
  console.log("User saved:", doc.name);
});

const User = mongoose.model("User", userSchema);
```

---

### 24. What are Mongoose Relationships (Referencing vs Embedding)?

MongoDB has two ways to model relationships:

#### **Embedding** — store related data inside the document:
```js
// Embed address inside user document
const userSchema = new mongoose.Schema({
  name: String,
  address: {          // embedded document
    street: String,
    city:   String,
    zip:    String,
  },
  // Embed small arrays directly
  tags: [String],                      // ["vip", "early-adopter"]
  recentOrders: [{                     // embedded array of objects
    orderId: mongoose.Schema.Types.ObjectId,
    total:   Number,
    date:    Date,
  }],
});

// Pros: One query to get all data, fast reads
// Cons: Document size limit (16MB), duplication if embedded data is shared
// Use for: Data only relevant to this document, small arrays, 1-1 or 1-few
```

#### **Referencing** — store related document's ID:
```js
const orderSchema = new mongoose.Schema({
  user: {
    type: mongoose.Schema.Types.ObjectId,
    ref: "User",    // reference to User model
    required: true,
  },
  products: [{
    product: { type: mongoose.Schema.Types.ObjectId, ref: "Product" },
    quantity: Number,
  }],
  total: Number,
});

// Populate — fetch referenced documents
const order = await Order
  .findById(id)
  .populate("user", "name email")          // only fetch name & email
  .populate("products.product", "name price"); // nested populate

// Pros: No data duplication, works for large or shared data
// Cons: Requires extra queries (or populate), more complex
// Use for: Many-to-many, shared data, large arrays
```

---

### 25. What are MongoDB Indexes?

Indexes in MongoDB speed up query performance by avoiding full collection scans.

```js
// Create indexes in Mongoose Schema
const userSchema = new mongoose.Schema({
  email:  { type: String, unique: true, index: true },
  name:   { type: String },
  age:    { type: Number },
  city:   { type: String },
  status: { type: String },
});

// Compound index
userSchema.index({ city: 1, age: 1 });

// Text index for full-text search
userSchema.index({ name: "text", bio: "text" });

// TTL index — auto-delete documents after expiry (great for sessions/OTPs)
const sessionSchema = new mongoose.Schema({
  token:     String,
  createdAt: { type: Date, default: Date.now, expires: "1h" }, // auto-delete after 1 hour
});

// Programmatically
await User.collection.createIndex({ email: 1 }, { unique: true });
await User.collection.createIndex({ city: 1, age: -1 });  // -1 = descending

// View indexes
await User.collection.indexes();

// Remove index
await User.collection.dropIndex("email_1");

// Run with explain to check if index is used
const result = await User.find({ email: "alice@mail.com" }).explain("executionStats");
console.log(result.executionStats.totalDocsExamined); // should be 1 (not all docs)
```

---

## 🔧 Section 4: Additional Database Concepts

---

### 26. What is Database Schema Design?

**Schema design** is the process of planning how to structure your data in a database. Good schema design is critical for performance and maintainability.

#### E-commerce Schema Example (SQL):
```sql
-- Users
CREATE TABLE users (
  id         SERIAL PRIMARY KEY,
  name       VARCHAR(100) NOT NULL,
  email      VARCHAR(255) UNIQUE NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Categories
CREATE TABLE categories (
  id     SERIAL PRIMARY KEY,
  name   VARCHAR(100) NOT NULL,
  parent_id INT REFERENCES categories(id)  -- self-referential for sub-categories
);

-- Products
CREATE TABLE products (
  id          SERIAL PRIMARY KEY,
  name        VARCHAR(200) NOT NULL,
  price       DECIMAL(10,2) NOT NULL CHECK (price >= 0),
  stock       INT DEFAULT 0,
  category_id INT REFERENCES categories(id),
  created_at  TIMESTAMPTZ DEFAULT NOW()
);

-- Orders
CREATE TABLE orders (
  id         SERIAL PRIMARY KEY,
  user_id    INT NOT NULL REFERENCES users(id),
  status     VARCHAR(20) DEFAULT 'pending'
             CHECK (status IN ('pending','processing','shipped','delivered','cancelled')),
  total      DECIMAL(10,2),
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Order Items (many-to-many: orders ↔ products)
CREATE TABLE order_items (
  id         SERIAL PRIMARY KEY,
  order_id   INT NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
  product_id INT NOT NULL REFERENCES products(id),
  quantity   INT NOT NULL CHECK (quantity > 0),
  price      DECIMAL(10,2) NOT NULL  -- snapshot price at time of purchase
);
```

---

### 27. What is an ER Diagram?

An **Entity-Relationship (ER) Diagram** is a visual blueprint of your database schema — showing entities (tables), attributes (columns), and relationships between them.

```
[User] ───has many──► [Order] ───contains──► [OrderItem] ───refers to──► [Product]
  |                                                                            |
  └── has one ──► [Address]                                        ───belongs to──► [Category]

Notations:
1    = one
*    = many
1..* = one to many
*..*  = many to many (needs a junction table)
```

**Relationship types:**
- **One-to-One (1:1)** — User ↔ Profile
- **One-to-Many (1:N)** — User → Orders (user has many orders)
- **Many-to-Many (N:M)** — Orders ↔ Products (via `order_items` junction table)

---

### 28. What is Caching and Redis?

**Caching** is storing frequently accessed data in a **fast, temporary storage** (like memory) so future requests can be served faster without hitting the database.

**Redis** is an in-memory **key-value store** commonly used as a cache, session store, or message broker.

```js
// Install: npm install redis
const redis = require("redis");
const client = redis.createClient({ url: process.env.REDIS_URL });
await client.connect();

// Basic operations
await client.set("user:1", JSON.stringify({ id: 1, name: "Alice" }));
const data = await client.get("user:1");
const user = JSON.parse(data);

// With expiry (TTL)
await client.setEx("otp:user123", 300, "482910"); // expires in 5 minutes

// Delete
await client.del("user:1");

// Caching pattern in Express
app.get("/users/:id", async (req, res) => {
  const cacheKey = `user:${req.params.id}`;

  // 1. Check cache first
  const cached = await redis.get(cacheKey);
  if (cached) {
    return res.json({ data: JSON.parse(cached), source: "cache" });
  }

  // 2. Cache miss — query DB
  const user = await User.findById(req.params.id);
  if (!user) return res.status(404).json({ error: "Not found" });

  // 3. Store in cache for next time (expire in 1 hour)
  await redis.setEx(cacheKey, 3600, JSON.stringify(user));

  res.json({ data: user, source: "database" });
});
```

**Redis use cases:**
- Caching DB query results
- Session/token storage
- Rate limiting counters
- Job queues
- Pub/Sub messaging
- Leaderboards (sorted sets)

---

### 29. What are Database Migrations?

**Migrations** are version-controlled files that describe changes to your database schema over time — allowing teams to evolve the schema in a controlled, repeatable way.

```js
// Using a migration tool (e.g., Knex.js, Sequelize, or Flyway)

// Migration file: 20240319_create_users_table.js
exports.up = async (knex) => {
  await knex.schema.createTable("users", (table) => {
    table.increments("id").primary();
    table.string("name", 100).notNullable();
    table.string("email", 255).unique().notNullable();
    table.string("password").notNullable();
    table.enum("role", ["user", "admin"]).defaultTo("user");
    table.timestamps(true, true); // created_at, updated_at
  });
};

exports.down = async (knex) => {
  await knex.schema.dropTableIfExists("users");
};

// Migration file: 20240320_add_phone_to_users.js
exports.up = async (knex) => {
  await knex.schema.alterTable("users", (table) => {
    table.string("phone", 20).nullable();
  });
};

exports.down = async (knex) => {
  await knex.schema.alterTable("users", (table) => {
    table.dropColumn("phone");
  });
};

// Commands:
// knex migrate:latest      — run all pending migrations
// knex migrate:rollback    — undo last migration
// knex migrate:status      — see which migrations have run
```

**Why use migrations:**
- Track schema changes in Git just like code
- Rollback changes if something goes wrong
- Collaborate safely in teams
- Automate deployments

---

### 30. What is Connection Pooling?

**Connection pooling** maintains a pool of **pre-established database connections** that can be reused, avoiding the overhead of creating a new connection for every request.

```js
// Without pooling — create/close connection per request — SLOW!
app.get("/users", async (req, res) => {
  const conn = await createConnection();  // expensive!
  const users = await conn.query("SELECT * FROM users");
  conn.close();                          // wasteful
  res.json(users);
});

// With pooling — connections are reused
const { Pool } = require("pg");

const pool = new Pool({
  host:     process.env.DB_HOST,
  database: process.env.DB_NAME,
  user:     process.env.DB_USER,
  password: process.env.DB_PASS,
  port:     5432,
  max: 20,              // max 20 connections in pool
  idleTimeoutMillis: 30000,  // close idle connections after 30s
  connectionTimeoutMillis: 2000, // timeout waiting for connection
});

app.get("/users", async (req, res) => {
  const { rows } = await pool.query("SELECT * FROM users");
  // connection is borrowed from pool and returned after query!
  res.json(rows);
});

// Mongoose also uses connection pooling internally
mongoose.connect(process.env.MONGO_URI, {
  maxPoolSize: 10,   // default is 5
});
```

---

### 31. What is Database Replication?

**Replication** is copying data from one database server (**primary/master**) to one or more other servers (**replicas/secondaries**) for:
- **High availability** — if primary goes down, replica takes over
- **Read scaling** — distribute read queries across replicas
- **Disaster recovery** — backup data in different locations

```
Primary (writes) ──replicates to──► Replica 1 (reads)
                ──replicates to──► Replica 2 (reads)
                ──replicates to──► Replica 3 (reads)
```

**MongoDB Replica Set example:**
```js
// Connect to a replica set
mongoose.connect(
  "mongodb://host1:27017,host2:27017,host3:27017/mydb?replicaSet=rs0"
);

// Reads can go to secondary replicas
const users = await User.find().read("secondaryPreferred");

// Writes always go to primary
await User.create({ name: "Alice" });
```

---

### 32. What is Sharding?

**Sharding** is splitting a large database **horizontally across multiple servers** — each shard holds a subset of the data.

```
All users:

Shard 1: users where _id starts with A-H  (machine 1)
Shard 2: users where _id starts with I-P  (machine 2)
Shard 3: users where _id starts with Q-Z  (machine 3)

Query → Router (mongos) → routes to correct shard(s)
```

**When to shard:**
- Dataset too large for one server (100GB+)
- Write throughput exceeds one server's capacity
- Single server RAM can't hold working dataset

**Sharding strategies:**
- **Range-based** — by value range (dates, IDs)
- **Hash-based** — consistent hash of key (even distribution)
- **Zone-based** — route by geographic region

---

### 33. What is SQL `UPDATE` and `DELETE` safely?

```sql
-- ⚠️ ALWAYS use WHERE with UPDATE and DELETE!

-- Safe UPDATE pattern
-- 1. First SELECT to verify what will be affected
SELECT * FROM users WHERE email = 'old@mail.com';
-- 2. Then UPDATE
UPDATE users SET email = 'new@mail.com' WHERE id = 42;

-- ❌ DANGEROUS — no WHERE clause updates ALL rows!
UPDATE users SET is_admin = true;  -- makes EVERYONE admin!
UPDATE products SET price = 0;     -- all prices become 0!

-- Safe DELETE pattern
BEGIN;
  DELETE FROM orders WHERE created_at < '2020-01-01';
  SELECT ROW_COUNT();  -- check how many rows affected
COMMIT;  -- only commit if the count looks right
-- Or ROLLBACK; if something looks wrong

-- Soft delete (safer — mark as deleted, don't actually remove)
ALTER TABLE users ADD COLUMN deleted_at TIMESTAMPTZ;

UPDATE users SET deleted_at = NOW() WHERE id = 42;

-- Then filter in queries:
SELECT * FROM users WHERE deleted_at IS NULL;
```

---

### 34. What is `EXPLAIN` and Query Optimization?

`EXPLAIN` shows the **execution plan** for a query — how the database will execute it, which indexes it uses, and estimated cost.

```sql
-- EXPLAIN — shows query plan
EXPLAIN SELECT * FROM users WHERE email = 'alice@mail.com';

-- EXPLAIN ANALYZE — actually runs query and shows real stats
EXPLAIN ANALYZE SELECT * FROM orders WHERE user_id = 1;

-- Example output indicators:
-- Seq Scan     → full table scan (no index) — SLOW for large tables
-- Index Scan   → using an index — FAST
-- Bitmap Scan  → using index for multiple rows
-- Nested Loop  → join algorithm
-- Hash Join    → join algorithm

-- Query optimization tips:
-- ✅ Add indexes on frequently filtered/joined columns
-- ✅ Use SELECT col1, col2 instead of SELECT * (fetch only needed data)
-- ✅ Avoid SELECT * in JOINs
-- ✅ Use LIMIT when you don't need all results
-- ✅ Use pagination (OFFSET/LIMIT or cursor-based)
-- ✅ Avoid using functions on indexed columns in WHERE
--    ❌  WHERE LOWER(email) = 'alice@mail.com'  (can't use index!)
--    ✅  WHERE email = 'alice@mail.com'     (lowercase at insert time)
-- ✅ Use EXISTS instead of IN for subqueries (usually faster)
-- ✅ Analyze slow queries with EXPLAIN ANALYZE
```

---

### 35. What is a View in SQL?

A **view** is a **saved SQL query** that acts like a virtual table. It doesn't store data itself — it retrieves data from the underlying tables when queried.

```sql
-- Create a view
CREATE VIEW active_users AS
  SELECT id, name, email, created_at
  FROM users
  WHERE is_active = true AND deleted_at IS NULL;

-- Use it like a table
SELECT * FROM active_users WHERE city = 'Mumbai';

-- More complex view
CREATE VIEW order_summary AS
  SELECT
    o.id          AS order_id,
    u.name        AS customer,
    u.email,
    COUNT(oi.id)  AS item_count,
    o.total,
    o.status,
    o.created_at
  FROM orders o
  JOIN users u      ON o.user_id = u.id
  JOIN order_items oi ON o.id = oi.order_id
  GROUP BY o.id, u.name, u.email;

-- Use the view
SELECT * FROM order_summary WHERE status = 'completed' ORDER BY total DESC;

-- Drop a view
DROP VIEW active_users;

-- Materialized View (PostgreSQL) — stores snapshot of query result (cached!)
CREATE MATERIALIZED VIEW monthly_stats AS
  SELECT DATE_TRUNC('month', created_at) AS month, SUM(total) AS revenue
  FROM orders GROUP BY month;

-- Refresh materialized view
REFRESH MATERIALIZED VIEW monthly_stats;
```

---

### 36. What is the difference between `TRUNCATE`, `DELETE`, and `DROP`?

| Command | What it does | Rollbackable | Structure | Speed |
|---|---|---|---|---|
| `DELETE [WHERE]` | Removes specific rows | ✅ Yes (in transaction) | Preserved | Slow (row-by-row) |
| `TRUNCATE` | Removes ALL rows | ❌ Usually not | Preserved | Very Fast |
| `DROP TABLE` | Deletes table + structure + data | ❌ No | Gone | Immediate |
| `DROP DATABASE` | Deletes entire database | ❌ No | Gone | Immediate |

```sql
-- DELETE — remove specific rows (with WHERE) or all rows (without)
DELETE FROM orders WHERE status = 'cancelled'; -- specific rows
DELETE FROM temp_table; -- all rows (slow, logged)

-- TRUNCATE — remove all rows fast (resets auto-increment!)
TRUNCATE TABLE temp_table;
TRUNCATE TABLE users RESTART IDENTITY CASCADE;  -- reset ID + cascade

-- DROP — removes table structure entirely
DROP TABLE IF EXISTS old_logs;
DROP DATABASE test_db;

-- Generally: DELETE for targeted removal, TRUNCATE to clear a table,
-- DROP to completely remove a table/database
```

---

### 37. What is a Stored Procedure?

A **stored procedure** is a **pre-compiled set of SQL statements** saved in the database that can be executed by name. Good for encapsulating complex logic on the DB side.

```sql
-- Create a stored procedure (PostgreSQL)
CREATE OR REPLACE FUNCTION transfer_money(
  from_account_id INT,
  to_account_id INT,
  amount DECIMAL
)
RETURNS VOID AS $$
BEGIN
  -- Check sufficient balance
  IF (SELECT balance FROM accounts WHERE id = from_account_id) < amount THEN
    RAISE EXCEPTION 'Insufficient balance';
  END IF;

  -- Debit
  UPDATE accounts SET balance = balance - amount WHERE id = from_account_id;
  -- Credit
  UPDATE accounts SET balance = balance + amount WHERE id = to_account_id;

  -- Log the transfer
  INSERT INTO transfer_log (from_id, to_id, amount) VALUES (from_account_id, to_account_id, amount);
END;
$$ LANGUAGE plpgsql;

-- Call the procedure
SELECT transfer_money(1, 2, 500.00);
```

---

### 38. What is the Difference Between `CHAR` and `VARCHAR`?

```sql
-- CHAR(n) — fixed-length string, always stores exactly n characters
-- Pads with spaces if shorter → wastes space for variable-length data
-- Slightly faster for truly fixed-length data (like country codes)
CREATE TABLE demo (
  country_code CHAR(2),   -- "IN" or "US" — always 2 chars
  gender       CHAR(1),   -- "M" or "F"
);

-- VARCHAR(n) — variable-length string, stores up to n characters
-- Only uses the space needed + small overhead → efficient for variable data
-- Most commonly used for text fields
CREATE TABLE users (
  name    VARCHAR(100),   -- up to 100 chars
  email   VARCHAR(255),   -- up to 255 chars
  bio     TEXT            -- unlimited length (PostgreSQL)
);

-- Comparison:
-- CHAR(10) storing "Hi"  → stores "Hi        " (8 spaces added) = 10 bytes
-- VARCHAR(10) storing "Hi" → stores "Hi"                        = 2 bytes + overhead
```

---

### 39. What is `DISTINCT` in SQL?

`DISTINCT` removes **duplicate values** from the result set.

```sql
-- Without DISTINCT — may return duplicates
SELECT city FROM users;
-- Mumbai, Delhi, Mumbai, Bangalore, Delhi, Mumbai

-- With DISTINCT — unique values only
SELECT DISTINCT city FROM users;
-- Bangalore, Delhi, Mumbai

-- Count unique values
SELECT COUNT(DISTINCT city) FROM users;  -- 3

-- DISTINCT on multiple columns — combination must be unique
SELECT DISTINCT city, country FROM users;

-- With ORDER BY
SELECT DISTINCT city FROM users ORDER BY city ASC;
```

---

### 40. What is a Self Join?

A **self join** is when a table is joined with **itself** — useful for hierarchical or comparative data within the same table.

```sql
-- employees table: id, name, manager_id (manager_id references employees.id)
-- Find each employee and their manager's name
SELECT
  e.name        AS employee,
  m.name        AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;

-- Result:
-- employee | manager
-- Alice    | Bob       ← Alice's manager is Bob
-- Bob      | NULL      ← Bob has no manager (CEO)
-- Charlie  | Alice

-- Find employees in the same department
SELECT
  e1.name AS employee1,
  e2.name AS employee2,
  e1.department
FROM employees e1
JOIN employees e2 ON e1.department = e2.department
  AND e1.id < e2.id  -- avoid duplicates and self-matching
ORDER BY e1.department;

-- Find products in the same category
SELECT p1.name AS product, p2.name AS related_product
FROM products p1
JOIN products p2 ON p1.category_id = p2.category_id
  AND p1.id != p2.id;
```

---

### 41. What is Database Backup and Recovery?

**Backup** = making copies of data so it can be restored if lost.
**Recovery** = restoring data from a backup after a failure.

```bash
# PostgreSQL backups

# Dump a database to a .sql file
pg_dump mydb > backup.sql

# Dump a specific table
pg_dump -t users mydb > users_backup.sql

# Restore from a .sql dump
psql mydb < backup.sql

# Binary format (faster for large DBs)
pg_dump -Fc mydb > backup.dump
pg_restore -d mydb backup.dump

# MongoDB backups

# Dump
mongodump --uri="mongodb://localhost/mydb" --out=./backup

# Restore
mongorestore --uri="mongodb://localhost/mydb" ./backup/mydb/
```

**Backup strategies:**
- **Full backup** — everything, every time (safe but large/slow)
- **Incremental backup** — only changes since last backup (fast, small)
- **Differential backup** — changes since last FULL backup
- **Point-in-time recovery** — restore to any specific moment using WAL/oplog

---

### 42. What are Common Database Interview Questions with Answers?

```sql
-- Q: Find the second highest salary
SELECT MAX(salary) FROM employees WHERE salary < (SELECT MAX(salary) FROM employees);
-- Or using LIMIT/OFFSET:
SELECT DISTINCT salary FROM employees ORDER BY salary DESC LIMIT 1 OFFSET 1;

-- Q: Find duplicate emails
SELECT email, COUNT(*) AS count
FROM users
GROUP BY email
HAVING COUNT(*) > 1;

-- Q: Delete duplicate rows (keep one)
DELETE FROM users
WHERE id NOT IN (
  SELECT MIN(id) FROM users GROUP BY email
);

-- Q: Find employees earning more than their manager
SELECT e.name AS employee, e.salary, m.name AS manager, m.salary AS manager_salary
FROM employees e
JOIN employees m ON e.manager_id = m.id
WHERE e.salary > m.salary;

-- Q: Running total
SELECT
  date,
  amount,
  SUM(amount) OVER (ORDER BY date) AS running_total
FROM transactions;

-- Q: Rank rows within groups (Window Functions)
SELECT
  name,
  department,
  salary,
  RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rank_in_dept
FROM employees;
```

---

### 43. What are Window Functions in SQL?

**Window functions** perform calculations across a **set of rows related to the current row**, without collapsing rows like GROUP BY does.

```sql
-- Syntax: function() OVER (PARTITION BY ... ORDER BY ...)

-- ROW_NUMBER — unique row number per partition
SELECT name, department, salary,
  ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS row_num
FROM employees;

-- RANK — rank with gaps for ties (1, 2, 2, 4)
-- DENSE_RANK — rank without gaps   (1, 2, 2, 3)
SELECT name, salary,
  RANK() OVER (ORDER BY salary DESC)       AS rank,
  DENSE_RANK() OVER (ORDER BY salary DESC) AS dense_rank
FROM employees;

-- LAG / LEAD — access previous/next row's value
SELECT date, amount,
  LAG(amount)  OVER (ORDER BY date) AS prev_amount,
  LEAD(amount) OVER (ORDER BY date) AS next_amount
FROM sales;

-- Running totals
SELECT date, amount,
  SUM(amount)   OVER (ORDER BY date) AS running_total,
  AVG(amount)   OVER (ORDER BY date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS 7day_avg
FROM sales;

-- NTILE — divide rows into N equal groups (percentiles)
SELECT name, salary,
  NTILE(4) OVER (ORDER BY salary) AS quartile  -- 1=bottom 25%, 4=top 25%
FROM employees;
```

---

### 44. What is a Cursor in SQL?

A **cursor** lets you iterate through a **result set row by row** — useful when you need to process each row individually (though usually set-based SQL is preferred for performance).

```sql
-- PostgreSQL cursor example
DO $$
DECLARE
  user_record RECORD;
  cur CURSOR FOR SELECT id, name, email FROM users WHERE NOT is_active;
BEGIN
  OPEN cur;
  LOOP
    FETCH cur INTO user_record;
    EXIT WHEN NOT FOUND;  -- exit when no more rows

    -- Process each row
    RAISE NOTICE 'Deactivating user: % (%)', user_record.name, user_record.email;
    -- (In practice, do this with a single UPDATE instead!)
  END LOOP;
  CLOSE cur;
END;
$$;
```

> **Best Practice:** Avoid cursors for large datasets — use set-based SQL (`UPDATE ... WHERE ...`) which is much faster. Use cursors only when you truly need row-by-row processing.

---

### 45. Quick SQL & MongoDB Cheat Sheet

```sql
-- ============ SQL CHEAT SHEET ============
-- Create table
CREATE TABLE users (id SERIAL PRIMARY KEY, name VARCHAR(100));

-- Insert
INSERT INTO users (name, email) VALUES ('Alice', 'alice@mail.com');

-- Select
SELECT * FROM users WHERE age > 18 ORDER BY name LIMIT 10;

-- Update
UPDATE users SET age = 26 WHERE id = 1;

-- Delete
DELETE FROM users WHERE id = 1;

-- Join
SELECT u.name, o.total FROM users u JOIN orders o ON u.id = o.user_id;

-- Aggregate
SELECT city, COUNT(*) as total, AVG(age) as avg_age FROM users GROUP BY city HAVING COUNT(*) > 5;
```

```js
// ============ MONGODB CHEAT SHEET ============
// Insert
await User.create({ name: "Alice", email: "alice@mail.com" });

// Find
await User.find({ age: { $gt: 18 } }).sort({ name: 1 }).limit(10);
await User.findOne({ email: "alice@mail.com" });
await User.findById("64abc123");

// Update
await User.findByIdAndUpdate(id, { $set: { age: 26 } }, { new: true });
await User.updateMany({ isActive: false }, { $set: { deletedAt: new Date() } });

// Delete
await User.findByIdAndDelete(id);
await User.deleteMany({ isActive: false });

// Aggregate
await User.aggregate([
  { $match: { age: { $gt: 18 } } },
  { $group: { _id: "$city", total: { $sum: 1 } } },
  { $sort: { total: -1 } }
]);
```

---

*End of Database Basics Reference*

---

### 46. What is a Database View?

**Definition:** A **View** is a virtual table based on the result-set of an SQL statement. It contains rows and columns just like a real table, but it **does not store data permanently**. It "looks" into the real tables.

**Why use it?**
-   **Security:** Hide sensitive columns (e.g., password_hash) from certain users.
-   **Simplicity:** Simplify complex joins into a single "table".
-   **Consistency:** Ensure the same logic is used across multiple reports.

```sql
CREATE VIEW active_users AS
SELECT id, name, email FROM users WHERE is_active = true;

SELECT * FROM active_users;
```

---

### 47. What is a Database Trigger?

**Definition:** A **Trigger** is a stored procedure that **automatically executes** ("fires") when a specific event occurs in the database (e.g., `INSERT`, `UPDATE`, or `DELETE`).

**Example:** Automatically log every time a user's email is updated.
```sql
CREATE TRIGGER log_email_change
AFTER UPDATE OF email ON users
FOR EACH ROW
EXECUTE FUNCTION log_change();
```

---

### 48. What is a Database Schema?

**Definition:** A **Schema** is the logical container or structure that defines how data is organized in a database. It includes tables, views, indexes, and relationships.

**In SQL (PostgreSQL/SQL Server):** A schema is like a folder *within* a database.
**In NoSQL (MongoDB):** A schema defines the shape of a document (usually enforced by Mongoose).

---

### 49. What are Aggregate Functions in SQL?

**Definition:** Aggregate functions perform a calculation on a set of values and return a **single value**. They are usually used with the `GROUP BY` clause.

**Common Functions:**
1.  **`COUNT()`**: Returns the number of rows.
2.  **`SUM()`**: Returns the total sum of a numeric column.
3.  **`AVG()`**: Returns the average value.
4.  **`MIN()`**: Returns the smallest value.
5.  **`MAX()`**: Returns the largest value.

```sql
SELECT department, COUNT(*), AVG(salary) 
FROM employees 
GROUP BY department;
```

---

### 50. SQL vs NoSQL – Which one to choose?

| Feature | SQL (Relational) | NoSQL (Non-Relational) |
|---|---|---|
| **Structure** | Fixed Schema (Tables/Rows) | Flexible Schema (Documents/JSON) |
| **Scaling** | Vertical (Bigger Server) | Horizontal (More Servers) |
| **ACID** | High (Strong Consistency) | Varies (High Availability) |
| **Best for** | Transactions, complex joins, financial data. | Real-time big data, content management, rapid prototyping. |

---

*End of Database Basics Reference*
