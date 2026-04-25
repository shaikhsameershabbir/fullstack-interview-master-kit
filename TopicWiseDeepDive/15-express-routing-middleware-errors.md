## 15) Express.js (routing, middleware, error handling)

### Definition (technical)
**Express** is a minimal Node.js web framework built on top of `http` that models request handling as a **middleware pipeline** (a chain of functions) with routing based on method/path and standardized error propagation via `next(err)` and error-handling middleware.

Express is popular because it gives you a clean mental model:

> An HTTP request flows through a pipeline of functions.

Teacher goal:
> You should be able to *predict* which middleware runs, in what order, and how errors flow — because most Express bugs are “pipeline misunderstandings.”

---

### Basic intuition (simple)
- Express is a thin layer over Node’s `http` server.
- Its core concept is **middleware**: functions that run in sequence for a request.
- Routing chooses which handler chain runs based on method + path.

### Internal working (engine level)
#### Middleware pipeline
Middleware signature:
- `(req, res, next)` for normal middleware
- `(err, req, res, next)` for error middleware

Express processes middleware in order:
1) global middleware (`app.use`)
2) route matching (`app.get('/x', ...)`)
3) error middleware when something throws/calls `next(err)`

#### Error handling
Key rules:
- Throwing inside async code won’t always be caught unless you return/await the Promise properly.
- Always have a centralized error handler to avoid leaking internal errors to clients.

#### Teacher walkthrough: “why middleware order matters”
Read this pipeline and predict what logs:

```js
app.use((req, res, next) => {
  console.log("A");
  next();
  console.log("A-after");
});

app.get("/x", (req, res) => {
  console.log("B");
  res.send("ok");
});
```

Expected behavior:
- `A` logs
- route handler logs `B`
- response is sent
- then `A-after` logs (because `next()` returns back to the middleware frame)

Teacher checkpoint:
> Middleware is just function calls. The “pipeline” is not magic — it’s ordering + `next()`.

### Edge cases
- Middleware order bugs are extremely common (auth after route, body parser after handler, etc.).
- Async errors: missing `await` can cause unhandled rejections.
- Double responses: calling `res.send` twice leads to header errors.

### Interview questions
- What is middleware and why is it powerful?
- How does Express decide which handler runs?
- How do you handle errors in async route handlers?

### Real-world usage (Node.js)
- Middleware is ideal for cross-cutting concerns:
  - auth
  - logging
  - request validation
  - rate limiting
  - tracing
- Prefer structured error responses and consistent status codes.

### Practice (do these in Node)
1) Write logging middleware that logs method + path + duration (time before/after `next()`).
2) Write auth middleware that blocks requests unless a header exists.
3) Create an async route that throws; make sure your centralized error middleware catches it and returns a safe response.

### Connections to other topics
- **HTTP lifecycle**: Express is built on top of Node `http`.
- **Async patterns**: route handlers often use async/await; error propagation matters.
- **Authentication**: auth is usually middleware.
