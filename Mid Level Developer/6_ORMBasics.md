# ORM Basics (Prisma & TypeORM) – Mid-Level Developer Interview Reference

---

## 🛠️ Section 1: ORM Architecture & Core Concepts

---

### 1. How does an ORM actually work (The Internal Layer)?

**Definition:** An ORM (Object-Relational Mapper) is a library that translates between **Objects** in your code (JavaScript/TypeScript) and **Rows** in a relational database.

**Internal Mechanics:**
1.  **Metadata:** The ORM reads your schema (Prisma) or decorators (TypeORM) to understand the table structure.
2.  **Query Builder:** It converts your method calls (e.g., `.findMany()`) into a raw SQL string (`SELECT * FROM ...`).
3.  **Hydration:** When the database returns raw rows, the ORM "hydrates" them into JavaScript objects, converting types (like `DATETIME` to `Date` objects) automatically.

---

### 2. Pros and Cons: ORM vs Raw SQL

| Feature | ORM | Raw SQL |
|---|---|---|
| **Speed of Dev** | **Fast.** Simple CRUD tasks take seconds. | Slow. You must write every query. |
| **Type Safety** | **High.** (Especially Prisma). | Low. (Unless using a manual mapper). |
| **Performance** | **Medium.** Might generate suboptimal queries. | **High.** You have total control. |
| **Portability** | High. Can switch from Postgres to MySQL easily. | Low. Query syntax differs between DBs. |

**Mid-Level Consensus:** Use an ORM for 90% of your app (standard business logic) and use **Raw SQL** for the 10% that requires high-performance complex aggregations.

---

### 3. What are the key components of a Prisma Schema?

1.  **DataSource:** Specifies the DB provider (postgres, mysql) and the connection URL.
2.  **Generator:** Specifies "Prisma Client" (the library you use in code).
3.  **Model:** Defines the Table structure, relationships (`@relation`), and constraints (`@unique`, `@id`).
4.  **Enums:** Defines fixed sets of values (e.g., `ROLE { USER, ADMIN }`).

---

### 4. Prisma Migrations: The Shadow Database

**Problem:** How does Prisma know what has changed in your schema without affecting your production data?

**The Solution:** During development, Prisma creates a temporary **"Shadow Database"**. 
-   It replays your entire migration history on the shadow DB.
-   It compares the shadow DB state with your current `schema.prisma`.
-   If they differ, it generates a new migration file.

---

### 5. What is "Prisma Drift"?

**Definition:** Drift occurs when the actual Database schema doesn't match your `schema.prisma`. 
-   **Cause:** Someone manually ran a `CREATE TABLE` command in the DB without using Prisma.
-   **Impact:** Prisma might fail or generate incorrect migrations.
-   **Fix:** Run `prisma db pull` to sync the schema file with the database, or manually fix the DB to match the schema.

---

### 6. Active Record vs Data Mapper Patterns

-   **Active Record (TypeORM / Sequelize):** The Model class itself has the save/delete methods. (`user.save()`). 
    -   *Pros:* Very intuitive and clean for small projects.
-   **Data Mapper (Prisma / TypeORM in mapper mode):** You use a "Repository" or "Client" to save the object. (`db.user.create({ data: user })`).
    -   *Pros:* Better for large systems as it separates business logic from database logic.

---

### 7. What is the "N+1 Query Problem"?

**Scenario:** You want to fetch 10 Users and their 100 Posts.
1.  **Query 1:** `SELECT * FROM users;` (Returns 10).
2.  **Queries 2-11:** For EACH user, run `SELECT * FROM posts WHERE user_id = X;`.
-   **Total:** 11 queries. (N users + 1 original query).

**Mid-Level Fix:** 
-   In Prisma: Use `include: { posts: true }`. Prisma automatically optimizes this into 1 or 2 queries using `IN` operators or Joins.

---

### 8. Eager Loading vs Lazy Loading

-   **Eager Loading:** Fetching all related data immediately in the first query.
-   **Lazy Loading:** Fetching related data only when you access the property (e.g., `user.posts`).
    -   *Danger:* Lazy loading is a major source of the N+1 problem because it hideously triggers hidden database calls in loops.

---

### 9. Handling "Transactions" in Prisma

**Interactive Transactions:**
```ts
const [user, profile] = await prisma.$transaction(async (tx) => {
  const u = await tx.user.create({ ... });
  const p = await tx.profile.create({ data: { userId: u.id } });
  return [u, p];
});
```
**Why use it?** If the Profile creation fails (e.g., validation error), the User creation is automatically rolled back, ensuring your database never has "Partial" data.

---

### 10. What is "Prisma Introspection"?

**Definition:** The process of generating a `schema.prisma` file from an **Existing Database**.
-   **Use Case:** You are moving an old project to Prisma. You don't want to write 100 models manually. You run `prisma db pull`, and Prisma scans your DB columns to build the schema for you.

---

### 11. Connection Pooling in Serverless environments

**Problem:** In AWS Lambda, every execution might spawn a new database connection. PostgreSQL has a hard limit on connections. 
-   **The Solution:** Use **Prisma Data Proxy** or **Accelerate**. They provide a "Cloud Pooler" that sits between your 1,000 Lambdas and your single Database.

---

### 12. ORM Middleware / Hooks

**Definition:** Functions that run automatically "Before" or "After" a query (e.g., `beforeInsert`, `afterUpdate`).

**Use Cases:**
1.  **Password Hashing:** Automatically hashing the password before saving it to the DB.
2.  **Auditing:** Automatically setting a `last_modified_by` field for every update.
3.  **Logging:** Sending the query duration to an analytics service for performance tracking.

---

### 13. Virtual Fields / Getters

**Problem:** You want a `fullName` property that combines `firstName` and `lastName`.
-   **Mid-Level Nuance:** You can define this in your ORM model, but you **cannot use it in a `WHERE` clause** because the field doesn't exist in the SQL database. You must compute it in JavaScript after fetching the data.

---

### 14. Polymorphic Associations

**Scenario:** A `Comment` can belong to either a `Post` or a `Video`.
-   **How ORMs handle it:** Usually via two columns: `commentable_id` and `commentable_type` (e.g., "POST" or "VIDEO").
-   **TypeORM Tip:** Use the `@Entity()` relation options to handle this. Prisma requires more manual work (Explicit relations) to maintain strict type safety.

---

### 15. TypeORM Subscribers vs Listeners

-   **Listeners:** Method decorators inside the entity (e.g., `@AfterInsert()`). Good for simple logic.
-   **Subscribers:** Separate classes that listen to all entities or specific ones.
    -   *Benefit:* Cleaner code as it separates the "Data definition" from the "Event Logic".

---

### 16. Using Raw SQL within an ORM

**When is it necessary?**
1.  **Complex Joins:** When the ORM-generated SQL is 500 lines long and slow.
2.  **Unsupported Features:** Using specialized Postgres features like `RECURSIVE CTE` or `WINDOW FUNCTIONS` which the ORM might not support yet.
3.  **Bulk Inserts:** Sometimes manual `COPY` commands (in Postgres) are 10x faster than ORM loops.

---

### 17. ORM Performance: Over-fetching

-   **Problem:** Calling `findMany()` returns **all columns** (e.g., 50 fields) even if you only need the `name`.
-   **Fix (Prisma):** Use `select: { id: true, name: true }`. This reduces the payload size and boosts performance.

---

### 18. Prisma Client Extensions (TS 4.7+)

**Definition:** Allows you to add custom "computed" methods to your models.
```ts
const prisma = new PrismaClient().$extends({
  model: {
    user: {
      async signUp(email: string) { ... }
    }
  }
});
// Now you can call prisma.user.signUp() directly!
```

---

### 19. Migrations: "Drift" and "Manual" Fixes

If you accidentally delete a migration file, Prisma gets confused. 
-   **Solution:** Use `prisma migrate resolve --applied <migration_name>` to tell Prisma "I manually fixed this, pretend this migration was already run".

---

### 20. Prisma vs TypeORM for Mid-Level Developers

-   **Choose Prisma:** If you want **Absolute Type Safety** and a clean schema-first approach. It generates the client based on the DB, which is much safer.
-   **Choose TypeORM:** If you want **Traditional Decorator-based OOP** or if you need to support very complex legacy databases that Prisma struggles with.

---

*Expansion Complete*
