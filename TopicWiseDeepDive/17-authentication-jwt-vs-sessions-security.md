## 17) Authentication (JWT vs Sessions, security basics)

### Definition (technical)
**Authentication** is the process of verifying an identity and establishing a security principal for a request; **authorization** determines allowed actions for that principal. In web systems this is commonly implemented using **sessions** (server-stored state referenced by a cookie/token) or **JWTs** (client-held signed tokens verified per request), plus secure password hashing and transport protections.

Authentication is where “backend engineering” meets “security reality.”

Teacher truth:
> A small auth mistake isn’t a “bug” — it’s a vulnerability.

So we’ll learn auth like a teacher: clear definitions, clear trade-offs, and the mental models that keep you safe.

---

### Basic intuition (simple)
- **Authentication**: “Who is this user?”
- **Authorization**: “What is this user allowed to do?”

Two common ways to remember a logged-in user:
- **Sessions**: server stores login state; client stores a session id
- **JWT**: client stores a signed token; server verifies it each request

Teacher checkpoint:
> Sessions are *stateful*. JWTs are *stateless verification* (mostly). The scaling and security trade-offs come from that difference.

---

### Internal working (engine level)
#### Sessions (stateful)
How it works:
1) user logs in
2) server creates a session record (memory/Redis/DB)
3) server sends session id to client (usually cookie)
4) each request: client sends session id → server looks up session → attaches user

Pros:
- easy logout/invalidation
- smaller tokens sent each request
- easy rotation of server-side state

Cons:
- needs shared store when scaling horizontally (Redis is common)
- session storage can grow (must be bounded/expired)

#### JWT (signed token, mostly stateless)
How it works:
1) user logs in
2) server signs a token containing claims (user id, roles, expiry)
3) client sends token on each request (header or cookie)
4) server verifies signature + expiry and reads claims

Pros:
- verification doesn’t require DB lookup (for basic identity)
- easy to distribute across many servers

Cons:
- logout/invalidation is hard without extra state (revocation lists, short expiry + refresh)
- token leakage is dangerous (tokens act like passwords)
- claims can become stale (roles changed but token still says old role)

Teacher checkpoint:
> JWT is not “more secure” than sessions. It’s a different trade-off.

#### Security basics (must-know)
1) **Password storage**
- never store plaintext
- use adaptive hashing: bcrypt/argon2/scrypt (not plain sha256)

2) **Transport security**
- use HTTPS
- treat tokens like passwords (don’t log them)

3) **Cookie flags**
- `HttpOnly`: JS can’t read it (helps against XSS token theft)
- `Secure`: only over HTTPS
- `SameSite`: helps reduce CSRF depending on flow

4) **Input validation**
- validate and sanitize inputs
- protect against injection and common attack patterns

---

### Edge cases (where real attacks happen)
- “Forever JWTs” (no expiry) are a common security bug.
- CSRF risk depends on how tokens are transported:
  - cookie-based auth is often CSRF-sensitive
  - header tokens are less CSRF-prone but more XSS-sensitive if stored in JS-accessible places
- Clock skew can cause “token just issued but considered expired” edge bugs.

---

### Interview questions
- Compare sessions vs JWT (trade-offs).
- How do you implement logout with JWT?
- Auth vs authorization: what’s the difference?
- How do you store passwords safely?
- What is CSRF and when is it a risk?

---

### Real-world usage (Node.js)
- Common production pattern:
  - short-lived access token
  - refresh token with rotation
- Implement auth as middleware:
  - verify identity
  - attach `req.user`
  - enforce authorization checks consistently
- Centralize security decisions (don’t scatter checks across random handlers).

---

### Practice (do these in Node)
1) Implement session auth with Redis-backed store; explain how scaling works.
2) Implement JWT auth with short expiry; add refresh flow (conceptually) and explain revocation strategy.
3) Write an authorization middleware: `requireRole("admin")`.

---

### Connections to other topics
- **15 Express**: auth is usually middleware.
- **13 Crypto / Buffer**: signing tokens and hashing passwords involve crypto primitives.
- **20 Scaling**: stateless vs stateful auth impacts load balancing and shared state.
