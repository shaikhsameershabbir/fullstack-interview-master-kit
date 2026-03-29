# Node.js & APIs – Complete Beginner Interview Reference

---

## 🟢 Section 1: Node.js Basics

---

### 1. What is Node.js?

**Node.js** is an **open-source, cross-platform JavaScript runtime environment** that allows you to run JavaScript **outside the browser** — on the server side. It is built on **Chrome's V8 JavaScript engine**.

**Key points:**
- Created by **Ryan Dahl** in 2009
- Uses an **event-driven, non-blocking I/O model** — making it lightweight and efficient
- Ideal for **data-intensive, real-time applications** (chat, APIs, streaming)
- Runs JavaScript on the **server** — same language on frontend AND backend

```
Browser (Chrome) → V8 Engine → runs JS
Node.js          → V8 Engine → runs JS (but on server/terminal!)
```

```js
// hello.js — runs with: node hello.js
console.log("Hello from Node.js!");

// Access system info (not possible in browser)
console.log(process.platform);  // "linux", "win32", "darwin"
console.log(process.version);   // "v18.17.0"
console.log(__dirname);         // current directory path
console.log(__filename);        // current file path
```

**Node.js is NOT:**
- A framework (Express is a framework built ON Node)
- Multi-threaded (it's single-threaded with an event loop)
- A language (JavaScript is the language)

---

### 2. Why use Node.js?

| Advantage | Explanation |
|---|---|
| **Same language** | JavaScript on frontend AND backend (full-stack JS) |
| **Non-blocking I/O** | Handles many requests without waiting (highly scalable) |
| **Fast** | V8 compiles JS to machine code; async ops don't block |
| **npm ecosystem** | Largest package registry in the world (~2 million packages) |
| **Real-time apps** | Perfect for chat, live notifications, streaming |
| **Microservices** | Lightweight — great for small, focused services |
| **Active community** | Huge ecosystem, lots of tutorials, libraries, support |

**Best use cases:**
- REST APIs and GraphQL APIs
- Real-time chat apps (Socket.io)
- Streaming applications (Netflix uses Node!)
- Microservices
- CLI tools
- BFF (Backend for Frontend)

**Not ideal for:**
- CPU-intensive tasks (image processing, ML, video encoding) — blocks the event loop
- Heavy computation (use Python, C++ for that)

---

### 3. What is Event-Driven Architecture in Node.js?

In **event-driven architecture**, the flow of the program is determined by **events** (user actions, messages, I/O completions) rather than a fixed sequence of instructions.

Node.js uses an **EventEmitter** pattern — objects can emit named events, and listeners (callbacks) respond to those events.

```js
const EventEmitter = require("events");

const emitter = new EventEmitter();

// Register a listener (subscriber)
emitter.on("userJoined", (name) => {
  console.log(`${name} joined the chat!`);
});

emitter.on("userJoined", (name) => {
  console.log(`Welcome, ${name}! 🎉`);
});

// Emit the event (trigger all listeners)
emitter.emit("userJoined", "Alice");
// Output:
// Alice joined the chat!
// Welcome, Alice! 🎉

// Listen only once
emitter.once("connect", () => console.log("Connected!"));

// Remove a listener
emitter.off("userJoined", listenerFn);

// List all listeners
emitter.listeners("userJoined");
```

**How Node.js uses this internally:**
- `http.Server` emits `request` events
- `fs.ReadStream` emits `data`, `end`, `error` events
- `net.Socket` emits `connect`, `data`, `close` events

---

### 4. What is Non-Blocking I/O?

**Blocking I/O:** The program **stops and waits** for an I/O operation (file read, DB query, network call) to complete before continuing.

**Non-Blocking I/O:** The program **initiates** an I/O operation and **immediately moves on** to handle other work. A callback/event notifies when the I/O is done.

```js
const fs = require("fs");

// ❌ BLOCKING — synchronous read (bad for servers)
const data = fs.readFileSync("bigfile.txt", "utf8");
console.log(data);
console.log("This runs AFTER file is fully read");

// ✅ NON-BLOCKING — asynchronous read (good for servers)
fs.readFile("bigfile.txt", "utf8", (err, data) => {
  if (err) throw err;
  console.log(data); // runs WHEN file is ready
});
console.log("This runs IMMEDIATELY, before file is read!");
```

**Why it matters:**
- A Node server can handle **thousands of simultaneous requests**
- While waiting for a DB query for Request A, it processes Request B, C, D...
- One slow DB call doesn't freeze the whole server

```
Traditional (blocking):     Node.js (non-blocking):
  Request 1 → wait (2s)       Request 1 → DB query (async)
  Request 1 → done            Request 2 → handled immediately
  Request 2 → wait (2s)       Request 3 → handled immediately
  ...                         Request 1 → DB responds → callback runs
```

---

### 5. What is npm?

**npm (Node Package Manager)** is the **default package manager for Node.js**. It comes pre-installed with Node.js and lets you:
- **Install** third-party packages (libraries/frameworks)
- **Publish** your own packages
- **Manage** project dependencies and scripts

```bash
# Check versions
node --version    # v20.x.x
npm --version     # 10.x.x

# Initialize a new project (creates package.json)
npm init          # interactive
npm init -y       # with all defaults

# Install a package
npm install express          # installs to node_modules, saves to dependencies
npm install --save-dev jest  # saves to devDependencies
npm install -g nodemon       # installs globally (available system-wide)

# Install all dependencies from package.json
npm install

# Remove a package
npm uninstall express

# Update packages
npm update
npm outdated  # shows what's outdated

# Run scripts defined in package.json
npm run start
npm run test
npm run build

# View package info
npm info express
npm list         # list installed packages

# Search
npm search lodash
```

**npm vs npx:**
- `npm` — installs and manages packages
- `npx` — executes a package without installing it globally (e.g., `npx create-react-app`)

---

### 6. What is `package.json`?

`package.json` is the **configuration file** for a Node.js project. It contains metadata about the project and manages dependencies and scripts.

```json
{
  "name": "my-app",
  "version": "1.0.0",
  "description": "A beginner Node.js app",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js",
    "test": "jest"
  },
  "dependencies": {
    "express": "^4.18.2",
    "mongoose": "^7.5.0"
  },
  "devDependencies": {
    "nodemon": "^3.0.1",
    "jest": "^29.6.4"
  },
  "keywords": ["api", "nodejs"],
  "author": "Alice",
  "license": "MIT",
  "engines": {
    "node": ">=18.0.0"
  }
}
```

**Key fields:**
| Field | Purpose |
|---|---|
| `name` | Package name (must be unique on npm) |
| `version` | Semantic versioning: `MAJOR.MINOR.PATCH` |
| `main` | Entry point file |
| `scripts` | Shell commands you can run with `npm run <name>` |
| `dependencies` | Required in production |
| `devDependencies` | Only needed during development (testing, building) |

**Version symbols:**
- `^4.18.2` — allows updates to `4.x.x` (minor & patch)
- `~4.18.2` — allows updates to `4.18.x` (patch only)
- `4.18.2` — exact version only

**`package-lock.json`** — automatically generated, locks exact versions for reproducible installs.

---

### 7. What is Express.js?

**Express.js** is a **minimal, fast, unopinionated web framework** for Node.js. It simplifies building web apps and REST APIs by providing:
- Routing (`GET`, `POST`, etc.)
- Middleware support
- Request/Response helpers
- Template engine support

```js
// Install: npm install express

const express = require("express");
const app = express();

// Middleware — parse JSON bodies
app.use(express.json());

// Routes
app.get("/", (req, res) => {
  res.send("Hello, World!");
});

app.get("/users", (req, res) => {
  res.json([{ id: 1, name: "Alice" }, { id: 2, name: "Bob" }]);
});

app.post("/users", (req, res) => {
  const { name } = req.body;
  res.status(201).json({ id: 3, name });
});

// Start server
const PORT = 3000;
app.listen(PORT, () => {
  console.log(`Server running at http://localhost:${PORT}`);
});
```

**Why Express over raw Node http?**
```js
// Raw Node.js http — verbose and manual
const http = require("http");
const server = http.createServer((req, res) => {
  if (req.method === "GET" && req.url === "/") {
    res.writeHead(200, { "Content-Type": "text/plain" });
    res.end("Hello!");
  }
});
server.listen(3000);

// Express — clean and readable ✅
app.get("/", (req, res) => res.send("Hello!"));
```

---

### 8. What is Middleware in Express?

**Middleware** is a function that has access to the **request (`req`)**, **response (`res`)**, and the **next middleware function (`next`)** in the request-response cycle.

Middleware runs **between** the request arriving and the response being sent.

```js
// Middleware signature
function myMiddleware(req, res, next) {
  // do something
  next(); // pass control to the next middleware/route
}

// 1. Application-level middleware (runs for all routes)
app.use((req, res, next) => {
  console.log(`${req.method} ${req.url} — ${new Date().toISOString()}`);
  next(); // MUST call next() or request hangs!
});

// 2. Route-level middleware (only for specific route)
app.get("/admin", isAuthenticated, (req, res) => {
  res.json({ admin: true });
});

function isAuthenticated(req, res, next) {
  const token = req.headers.authorization;
  if (!token) return res.status(401).json({ error: "Unauthorized" });
  next();
}

// 3. Error-handling middleware (4 params — must come LAST)
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: err.message });
});

// 4. Built-in middleware
app.use(express.json());                          // parse JSON body
app.use(express.urlencoded({ extended: true }));  // parse form data
app.use(express.static("public"));               // serve static files

// 5. Third-party middleware
const cors = require("cors");
const morgan = require("morgan");
app.use(cors());          // enable CORS
app.use(morgan("dev"));   // HTTP request logger
```

**Middleware order matters** — they run top to bottom in the order defined.

---

### 9. What is the `req` and `res` Object?

**`req` (Request)** — contains everything about the incoming HTTP request.
**`res` (Response)** — used to send the HTTP response back to the client.

```js
app.get("/users/:id", (req, res) => {

  // --- req (Request) ---
  req.method;           // "GET"
  req.url;              // "/users/42?active=true"
  req.path;             // "/users/42"
  req.params;           // { id: "42" } — URL route parameters
  req.query;            // { active: "true" } — query string ?key=val
  req.body;             // { name: "Alice" } — parsed request body (needs middleware)
  req.headers;          // { authorization: "Bearer ...", "content-type": "..." }
  req.headers["content-type"]; // "application/json"
  req.ip;               // "127.0.0.1"
  req.cookies;          // (needs cookie-parser middleware)

  // --- res (Response) ---
  res.status(200);                        // set status code
  res.send("Hello!");                     // send text / HTML
  res.json({ id: 1, name: "Alice" });     // send JSON (auto sets Content-Type)
  res.status(201).json({ created: true }); // chain status + json
  res.sendFile("/path/to/file.pdf");      // send a file
  res.redirect("/login");                 // redirect to another URL
  res.set("X-Custom-Header", "value");    // set response header
  res.cookie("token", "abc123");          // set a cookie
  res.end();                              // end response with no data
});
```

---

### 10. What is Routing?

**Routing** is how an application determines what to do with an incoming request based on the **HTTP method** and **URL path**.

```js
const express = require("express");
const app = express();
app.use(express.json());

// Basic routes — method + path + handler
app.get("/",          (req, res) => res.send("Home"));
app.get("/about",     (req, res) => res.send("About"));
app.post("/users",    (req, res) => res.status(201).json({ created: true }));
app.put("/users/:id", (req, res) => res.json({ updated: req.params.id }));
app.delete("/users/:id", (req, res) => res.json({ deleted: req.params.id }));

// Route Parameters — :param captures dynamic values
app.get("/users/:id/posts/:postId", (req, res) => {
  const { id, postId } = req.params;
  res.json({ userId: id, postId });
});

// Query Strings — /search?q=node&limit=10
app.get("/search", (req, res) => {
  const { q, limit = 10 } = req.query;
  res.json({ query: q, limit });
});

// Route grouping with express.Router (best practice for large apps)
const userRouter = express.Router();

userRouter.get("/",      (req, res) => res.json({ users: [] }));
userRouter.get("/:id",   (req, res) => res.json({ id: req.params.id }));
userRouter.post("/",     (req, res) => res.status(201).json({ created: true }));
userRouter.put("/:id",   (req, res) => res.json({ updated: true }));
userRouter.delete("/:id",(req, res) => res.json({ deleted: true }));

// Mount router at /users prefix
app.use("/users", userRouter);
// Results in: GET /users, GET /users/:id, POST /users, etc.
```

---

## 🌐 Section 2: REST APIs

---

### 11. What is a REST API?

**REST (Representational State Transfer)** is an **architectural style** for designing networked applications. A **REST API** is an API that follows REST principles to allow communication between a client and server over HTTP.

**6 REST Principles:**
1. **Client-Server** — frontend and backend are separate; communicate via HTTP
2. **Stateless** — each request contains ALL info needed; server stores NO session
3. **Cacheable** — responses can be cached for performance
4. **Uniform Interface** — standardized URL structure, HTTP methods, status codes
5. **Layered System** — client doesn't know if it's talking to server, cache, or load balancer
6. **Code on Demand** (optional) — server can send executable code (JavaScript)

```
Client (React, Mobile, etc.)
         ↕ HTTP Request (GET /users/1)
REST API Server (Express + Node.js)
         ↕ Query
Database (MongoDB, PostgreSQL)
```

**REST API resource example:**
```
Resource: /users

GET    /users         → Get all users
POST   /users         → Create a new user
GET    /users/:id     → Get one user
PUT    /users/:id     → Replace a user completely
PATCH  /users/:id     → Update specific fields of a user
DELETE /users/:id     → Delete a user
```

**RESTful response:**
```json
{
  "status": "success",
  "data": {
    "id": 1,
    "name": "Alice",
    "email": "alice@example.com"
  }
}
```

---

### 12. What are HTTP Methods?

HTTP methods (also called **HTTP verbs**) define the **action** to perform on a resource.

| Method | Action | Has Body | Idempotent | Safe |
|---|---|---|---|---|
| `GET` | Read / fetch data | ❌ | ✅ | ✅ |
| `POST` | Create new resource | ✅ | ❌ | ❌ |
| `PUT` | Replace entire resource | ✅ | ✅ | ❌ |
| `PATCH` | Update part of resource | ✅ | ✅ | ❌ |
| `DELETE` | Delete a resource | ❌/✅ | ✅ | ❌ |
| `HEAD` | Like GET but no body | ❌ | ✅ | ✅ |
| `OPTIONS` | Lists allowed methods | ❌ | ✅ | ✅ |

- **Idempotent** = calling it multiple times has the same result as calling it once
- **Safe** = does NOT change server state

```js
// Express handles all HTTP methods
app.get("/resource", handler);
app.post("/resource", handler);
app.put("/resource/:id", handler);
app.patch("/resource/:id", handler);
app.delete("/resource/:id", handler);
app.options("/resource", handler);
```

---

### 13. What is a GET Request?

`GET` is used to **retrieve / read** data from the server. It should **never modify** data.

```js
// Client side (fetch)
const response = await fetch("https://api.example.com/users");
const users = await response.json();

// With query parameters: GET /users?role=admin&limit=10
const res = await fetch("/users?role=admin&limit=10");

// Server side (Express)
app.get("/users", async (req, res) => {
  try {
    const { role, limit = 10 } = req.query;
    const users = await User.find({ role }).limit(Number(limit));
    res.json({ status: "success", data: users });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

app.get("/users/:id", async (req, res) => {
  const user = await User.findById(req.params.id);
  if (!user) return res.status(404).json({ error: "User not found" });
  res.json({ data: user });
});
```

**GET rules:**
- Parameters go in the **URL** (path params or query string) — NOT in body
- Responses should be **cacheable**
- No side effects — calling it multiple times is safe

---

### 14. What is a POST Request?

`POST` is used to **create a new resource** on the server. The data is sent in the **request body**.

```js
// Client side (fetch)
const response = await fetch("/users", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ name: "Alice", email: "alice@mail.com" })
});
const newUser = await response.json();

// Server side (Express)
app.post("/users", async (req, res) => {
  try {
    const { name, email, password } = req.body;

    // Validate
    if (!name || !email) {
      return res.status(400).json({ error: "Name and email are required" });
    }

    // Create in DB
    const user = await User.create({ name, email, password });

    // Respond with 201 Created
    res.status(201).json({
      status: "success",
      data: user
    });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});
```

**POST rules:**
- Returns **201 Created** on success
- **NOT idempotent** — calling it twice creates two resources
- Data sent in **request body**
- Requires `Content-Type: application/json` header

---

### 15. What is `PUT` vs `PATCH`?

Both update a resource, but differ in **how much** they update:

| | `PUT` | `PATCH` |
|---|---|---|
| Updates | **Entire resource** (full replacement) | **Partial resource** (specific fields only) |
| Missing fields | Set to `null`/removed | Left unchanged |
| Idempotent | ✅ Yes | ✅ Yes |
| Body required | ✅ (full object) | ✅ (only changed fields) |

```js
// Example resource:
// { id: 1, name: "Alice", email: "alice@mail.com", age: 25 }

// PUT — sends the FULL updated object
// PUT /users/1
// Body: { name: "Alice Updated", email: "new@mail.com", age: 25 }
// → Replaces the entire document (age must be included or it's lost!)
app.put("/users/:id", async (req, res) => {
  const user = await User.findByIdAndReplace(req.params.id, req.body, { new: true });
  res.json({ data: user });
});

// PATCH — sends ONLY the fields you want to change
// PATCH /users/1
// Body: { name: "Alice Updated" }
// → Only name changes, email and age remain untouched ✅
app.patch("/users/:id", async (req, res) => {
  const user = await User.findByIdAndUpdate(
    req.params.id,
    { $set: req.body },   // only update sent fields
    { new: true }
  );
  res.json({ data: user });
});
```

> **Best Practice:** Use `PATCH` for partial updates (most common in practice). Use `PUT` only when you're replacing the full resource.

---

### 16. What is the DELETE Method?

`DELETE` is used to **remove a resource** from the server.

```js
// Client side
await fetch("/users/1", { method: "DELETE" });

// Server side (Express)
app.delete("/users/:id", async (req, res) => {
  try {
    const user = await User.findByIdAndDelete(req.params.id);

    if (!user) {
      return res.status(404).json({ error: "User not found" });
    }

    // Common patterns for response:
    res.status(200).json({ message: "User deleted successfully" });
    // OR return 204 No Content (no body)
    res.status(204).send();
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// Soft delete (mark as deleted, don't remove from DB)
app.delete("/users/:id", async (req, res) => {
  await User.findByIdAndUpdate(req.params.id, { deletedAt: new Date() });
  res.json({ message: "User deactivated" });
});
```

**DELETE response options:**
- `200 OK` + message body (most common)
- `204 No Content` — success, no body
- `404 Not Found` — if resource doesn't exist

---

### 17. What are HTTP Status Codes?

HTTP status codes indicate the **result of an HTTP request**. They are grouped in ranges:

#### 2xx — Success
| Code | Name | When to use |
|---|---|---|
| `200` | OK | Successful GET, PUT, PATCH, DELETE |
| `201` | Created | Successful POST (new resource created) |
| `204` | No Content | Success but no response body (e.g., DELETE) |

#### 3xx — Redirection
| Code | Name | When to use |
|---|---|---|
| `301` | Moved Permanently | URL permanently changed |
| `302` | Found | Temporary redirect |
| `304` | Not Modified | Cached response is valid |

#### 4xx — Client Errors
| Code | Name | When to use |
|---|---|---|
| `400` | Bad Request | Invalid data / malformed request |
| `401` | Unauthorized | Not authenticated (no/invalid token) |
| `403` | Forbidden | Authenticated but not authorized |
| `404` | Not Found | Resource doesn't exist |
| `409` | Conflict | Duplicate resource (e.g., email already exists) |
| `422` | Unprocessable Entity | Validation error |
| `429` | Too Many Requests | Rate limit exceeded |

#### 5xx — Server Errors
| Code | Name | When to use |
|---|---|---|
| `500` | Internal Server Error | Unexpected server error |
| `502` | Bad Gateway | Upstream server returned invalid response |
| `503` | Service Unavailable | Server is down/overloaded |

```js
// Express examples
res.status(200).json({ data: users });
res.status(201).json({ data: newUser });
res.status(400).json({ error: "Name is required" });
res.status(401).json({ error: "Please log in first" });
res.status(403).json({ error: "Admin access required" });
res.status(404).json({ error: "User not found" });
res.status(500).json({ error: "Internal server error" });
```

---

### 18. What is JSON?

**JSON (JavaScript Object Notation)** is a **lightweight text format** for data exchange between client and server. It is the standard format for REST APIs.

```json
{
  "id": 1,
  "name": "Alice",
  "age": 25,
  "isAdmin": false,
  "address": {
    "city": "Mumbai",
    "zip": "400001"
  },
  "skills": ["JavaScript", "Node.js", "React"],
  "metadata": null
}
```

**JSON rules:**
- Keys must be **double-quoted strings**
- Values can be: string, number, boolean, null, array, object
- **No** functions, `undefined`, `Date`, `Symbol`
- **No** trailing commas
- **No** comments

```js
// In Node.js / Express
const obj = { name: "Alice", age: 25 };

// Convert JS object to JSON string
const jsonStr = JSON.stringify(obj);
// '{"name":"Alice","age":25}'

// Convert JSON string to JS object
const parsed = JSON.parse(jsonStr);
parsed.name; // "Alice"

// Express auto-handles this:
app.use(express.json()); // parses incoming JSON body into req.body
res.json(obj);           // converts obj to JSON and sends it
```

---

### 19. What is an API Endpoint?

An **API endpoint** is a specific **URL** that points to a particular resource or function in an API. It is the point of communication between the client and server.

```
Base URL: https://api.myapp.com

Endpoints:

GET    https://api.myapp.com/users          → List all users
POST   https://api.myapp.com/users          → Create user
GET    https://api.myapp.com/users/42       → Get user with ID 42
PATCH  https://api.myapp.com/users/42       → Update user 42
DELETE https://api.myapp.com/users/42       → Delete user 42
GET    https://api.myapp.com/users/42/posts → Get all posts by user 42
POST   https://api.myapp.com/auth/login     → Login
POST   https://api.myapp.com/auth/register  → Register
GET    https://api.myapp.com/products?category=shoes&limit=20
```

**Endpoint naming conventions (REST best practices):**
- Use **nouns**, not verbs: `/users` not `/getUsers`
- Use **plural** names: `/users` not `/user`
- Use **lowercase** and **hyphens**: `/blog-posts` not `/blogPosts`
- Nest related resources: `/users/:id/orders`
- Don't nest more than 2 levels deep

---

### 20. What is API Versioning?

**API versioning** is a strategy to **manage changes** to an API without breaking existing clients. When you change an API, old clients should still work.

**Methods of versioning:**

#### 1. URL path versioning (most common):
```
GET /api/v1/users
GET /api/v2/users
```

```js
const v1Router = require("./routes/v1");
const v2Router = require("./routes/v2");

app.use("/api/v1", v1Router);
app.use("/api/v2", v2Router);
```

#### 2. Query parameter versioning:
```
GET /api/users?version=1
GET /api/users?version=2
```

#### 3. Header versioning:
```
GET /api/users
Headers: { "API-Version": "2" }
```

#### 4. Content negotiation (Accept header):
```
GET /api/users
Headers: { "Accept": "application/vnd.myapp.v2+json" }
```

**When to version your API:**
- Breaking changes (removing/renaming fields, changing response structure)
- Changes to authentication
- Business logic changes that affect existing clients

**Best practice:** Always include versioning from day one!

---
## 📁 Section 3: Node.js Core Modules & Additional Concepts

---

### 21. What are Node.js Core (Built-in) Modules?

Node.js ships with many **built-in modules** you can use without installing anything. You import them with `require()`.

```js
// Common built-in modules
const fs      = require("fs");       // file system operations
const path    = require("path");     // file path utilities
const http    = require("http");     // create HTTP server
const https   = require("https");   // HTTPS server
const os      = require("os");       // operating system info
const events  = require("events");  // EventEmitter
const crypto  = require("crypto");  // hashing, encryption
const url     = require("url");      // URL parsing
const util    = require("util");     // utility functions
const stream  = require("stream");  // streaming data
const buffer  = require("buffer");  // binary data
const querystring = require("querystring"); // parse query strings
const process  = require("process"); // already global in Node
```

```js
// os module example
const os = require("os");
os.platform();   // "linux", "darwin", "win32"
os.cpus();       // array of CPU core details
os.totalmem();   // total RAM in bytes
os.freemem();    // free RAM in bytes
os.homedir();    // "/home/alice"
os.hostname();   // machine hostname

// path module example
const path = require("path");
path.join(__dirname, "public", "index.html"); // safe path joining
path.resolve("./folder", "file.js");          // absolute path
path.extname("photo.jpg");                    // ".jpg"
path.basename("/users/alice/file.txt");        // "file.txt"
path.dirname("/users/alice/file.txt");         // "/users/alice"
```

---

### 22. What is the `fs` Module?

The **`fs` (File System)** module lets you interact with the server's file system — read, write, delete, rename files and directories.

```js
const fs = require("fs");
const { promisify } = require("util");

// --- SYNCHRONOUS (blocks execution — avoid in production) ---
const data = fs.readFileSync("hello.txt", "utf8");
fs.writeFileSync("output.txt", "Hello, World!");

// --- ASYNCHRONOUS CALLBACK style (traditional Node.js) ---
fs.readFile("hello.txt", "utf8", (err, data) => {
  if (err) throw err;
  console.log(data);
});

fs.writeFile("output.txt", "Hello!", (err) => {
  if (err) throw err;
  console.log("File written!");
});

// --- PROMISES style (modern, recommended) ---
const fs = require("fs/promises");

async function readAndWrite() {
  try {
    const content = await fs.readFile("input.txt", "utf8");
    await fs.writeFile("output.txt", content.toUpperCase());
    console.log("Done!");
  } catch (err) {
    console.error(err);
  }
}

// --- Other fs operations ---
fs.promises.appendFile("log.txt", "New line\n");  // append to file
fs.promises.unlink("file.txt");                   // delete file
fs.promises.rename("old.txt", "new.txt");         // rename/move
fs.promises.mkdir("myFolder", { recursive: true }); // create directory
fs.promises.readdir("./");                        // list directory contents
fs.promises.stat("file.txt");                     // file info (size, dates)
```

---

### 23. What is the `path` Module?

The **`path`** module provides utilities for working with file and directory paths in a cross-platform way.

```js
const path = require("path");

// Joining paths (works on all OS - handles / vs \ automatically)
path.join("/users", "alice", "documents", "file.txt");
// "/users/alice/documents/file.txt"

path.join(__dirname, "routes", "user.js");
// Absolute path to routes/user.js in current project

// Resolve to absolute path
path.resolve("./src/index.js");
// "/home/alice/myproject/src/index.js"

// Extract parts
path.extname("photo.jpg");         // ".jpg"
path.extname("archive.tar.gz");    // ".gz"
path.basename("path/to/file.txt"); // "file.txt"
path.basename("file.txt", ".txt"); // "file" (remove extension)
path.dirname("/users/alice/file"); // "/users/alice"

// Parse a path into parts
path.parse("/users/alice/docs/file.txt");
// {
//   root: "/",
//   dir: "/users/alice/docs",
//   base: "file.txt",
//   ext: ".txt",
//   name: "file"
// }

// Build path from parts
path.format({ dir: "/users/alice", base: "file.txt" });
// "/users/alice/file.txt"

// Check if path is absolute
path.isAbsolute("/users/alice"); // true
path.isAbsolute("./relative");  // false
```

**Useful globals in Node.js:**
```js
__dirname   // absolute path to current file's directory
__filename  // absolute path to current file
```

---

### 24. What is `process` in Node.js?

The **`process`** object is a **global** in Node.js (no need to require it). It provides info about the current Node.js process and lets you interact with it.

```js
// Environment & system info
process.env.NODE_ENV         // "development" | "production" | "test"
process.env.PORT             // "3000" (from environment variables)
process.version              // "v20.0.0"
process.platform             // "linux", "darwin", "win32"
process.pid                  // process ID (number)
process.uptime()             // seconds process has been running
process.memoryUsage()        // { rss, heapTotal, heapUsed, external }

// Command-line arguments (node app.js --port=3000)
process.argv                 // [ "node", "app.js", "--port=3000" ]
process.argv[2]              // "--port=3000"

// Exit the process
process.exit(0);             // clean exit (0 = success)
process.exit(1);             // exit with error

// Listen for process events
process.on("exit", (code) => {
  console.log(`Exiting with code: ${code}`);
});

process.on("uncaughtException", (err) => {
  console.error("Unhandled error:", err);
  process.exit(1);
});

process.on("unhandledRejection", (reason) => {
  console.error("Unhandled promise rejection:", reason);
});

// Standard I/O streams
process.stdin.on("data", (data) => console.log(data.toString()));
process.stdout.write("Hello without newline");
process.stderr.write("Error message");
```

---

### 25. What are Environment Variables? How to use `.env`?

**Environment variables** store **configuration values** (secrets, URLs, ports) outside the codebase, so they differ between environments (dev, staging, prod) without code changes.

**Never hardcode secrets like API keys or DB passwords in code!**

```bash
# Set env vars in terminal (temporary)
export PORT=3000
export DB_URL=mongodb://localhost/mydb

# Or in .env file (permanent for the project)
```

```ini
# .env file (in project root — add to .gitignore!)
PORT=3000
NODE_ENV=development
DB_URL=mongodb://localhost:27017/mydb
JWT_SECRET=myverysecretkey123
API_KEY=sk-abc123xyz
EMAIL_HOST=smtp.gmail.com
```

```js
// Install: npm install dotenv
require("dotenv").config(); // loads .env into process.env

// Access env variables
const PORT = process.env.PORT || 3000;
const DB_URL = process.env.DB_URL;
const JWT_SECRET = process.env.JWT_SECRET;

console.log(process.env.NODE_ENV); // "development"

// In Express
app.listen(PORT, () => console.log(`Server on port ${PORT}`));
```

```
# .gitignore — ALWAYS ignore .env files!
node_modules/
.env
.env.local
.env.production
```

**Production:** Set environment variables through hosting platform (Heroku config vars, Railway variables, AWS Parameter Store, etc.)

---

### 26. What is `nodemon`?

**`nodemon`** is a development tool that **automatically restarts the Node.js server** whenever it detects file changes — so you don't have to manually stop and restart.

```bash
# Install as dev dependency
npm install --save-dev nodemon

# Or install globally
npm install -g nodemon
```

```json
// package.json scripts
{
  "scripts": {
    "start": "node index.js",          // production
    "dev": "nodemon index.js",          // development with auto-restart
    "dev:debug": "nodemon --inspect index.js"
  }
}
```

```bash
npm run dev   # starts with nodemon — auto-restarts on file save
```

```js
// nodemon.json — custom configuration
{
  "watch": ["src"],           // watch this directory
  "ext": "js,json",           // watch these file extensions
  "ignore": ["src/**/*.test.js"],  // ignore test files
  "delay": 1000               // wait 1s before restarting
}
```

---

### 27. What is the difference between `require()` and `import`?

| Feature | `require()` (CommonJS) | `import` (ES Modules) |
|---|---|---|
| Standard | Node.js default (older) | ES6 standard (modern) |
| Loading | Synchronous | Asynchronous |
| Dynamic | ✅ (can use in if blocks) | ❌ (static, top-level only) |
| Default in Node | ✅ `.js` files | Requires `"type": "module"` or `.mjs` |
| Tree shaking | ❌ | ✅ (bundlers can remove unused) |

```js
// CommonJS (require) — default in Node.js
const express = require("express");
const { readFile } = require("fs/promises");
const myModule = require("./myModule");

// Dynamic require (possible)
if (condition) {
  const thing = require("./thing");
}

// ES Modules (import) — modern standard
import express from "express";
import { readFile } from "fs/promises";
import myModule from "./myModule.js";  // .js extension required

// Dynamic import (returns Promise)
const module = await import("./heavyModule.js");
```

**Using ES Modules in Node.js:**
```json
// package.json
{
  "type": "module"  // enables import/export in .js files
}
```

---

### 28. What is CORS? How to handle it in Express?

**CORS (Cross-Origin Resource Sharing)** is a browser security mechanism that **blocks web pages from making requests to a different domain** than the one that served the page.

```
Your Frontend: http://localhost:3000
Your API:      http://localhost:5000

Browser blocks: http://localhost:3000 → http://localhost:5000  ← CORS error!
```

**Origins are different if port, protocol, or domain differs.**

```js
// Install: npm install cors
const cors = require("cors");

// Allow ALL origins (dev only — not for production)
app.use(cors());

// Allow specific origin
app.use(cors({
  origin: "http://localhost:3000",     // single origin
}));

// Allow multiple origins
app.use(cors({
  origin: ["http://localhost:3000", "https://myapp.com"],
  methods: ["GET", "POST", "PUT", "PATCH", "DELETE"],
  allowedHeaders: ["Content-Type", "Authorization"],
  credentials: true,   // allow cookies/auth headers
}));

// Apply CORS to specific routes only
app.get("/public", cors(), (req, res) => res.json({ data: "public" }));
```

**CORS only matters in browsers** — tools like Postman, curl, or server-to-server calls don't have CORS restrictions.

---

### 29. What is Authentication vs Authorization?

| | Authentication | Authorization |
|---|---|---|
| Question | **"Who are you?"** | **"What can you do?"** |
| Process | Verify identity (login) | Check permissions (access control) |
| Example | Checking username + password | Checking if user is admin |
| Happens | First | After authentication |
| Common tech | JWT, Sessions, OAuth | Roles, Permissions, ACL |

```js
// Authentication — verify who the user is
app.post("/auth/login", async (req, res) => {
  const { email, password } = req.body;
  const user = await User.findOne({ email });

  if (!user || !await bcrypt.compare(password, user.password)) {
    return res.status(401).json({ error: "Invalid credentials" });
  }

  const token = jwt.sign({ id: user._id, role: user.role }, process.env.JWT_SECRET);
  res.json({ token });
});

// Authentication middleware — verify token on protected routes
function authenticate(req, res, next) {
  const token = req.headers.authorization?.split(" ")[1]; // "Bearer <token>"
  if (!token) return res.status(401).json({ error: "No token provided" });

  try {
    req.user = jwt.verify(token, process.env.JWT_SECRET);
    next();
  } catch {
    res.status(401).json({ error: "Invalid token" });
  }
}

// Authorization middleware — check permissions
function authorize(...roles) {
  return (req, res, next) => {
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({ error: "Access forbidden" });
    }
    next();
  };
}

// Usage
app.get("/profile", authenticate, (req, res) => {
  res.json({ user: req.user }); // any authenticated user
});

app.delete("/users/:id", authenticate, authorize("admin"), (req, res) => {
  // only admins can delete users
});
```

---

### 30. What is JWT (JSON Web Token)?

**JWT** is a compact, self-contained token used for **securely transmitting information** between parties. Widely used for **stateless authentication** in REST APIs.

**JWT Structure:** `header.payload.signature`
```
eyJhbGciOiJIUzI1NiJ9.eyJpZCI6MSwidXNlciI6IkFsaWNlIn0.abc123signature
     HEADER                        PAYLOAD                   SIGNATURE
```

```js
// Install: npm install jsonwebtoken
const jwt = require("jsonwebtoken");

const SECRET = process.env.JWT_SECRET; // keep this safe!

// 1. Create (sign) a token — on login
const token = jwt.sign(
  { id: user._id, email: user.email, role: "admin" },  // payload
  SECRET,                                               // secret key
  { expiresIn: "7d" }                                   // options
);
// Returns a string token — send to client

// 2. Verify a token — on protected requests
try {
  const decoded = jwt.verify(token, SECRET);
  console.log(decoded); // { id: 1, email: "...", role: "admin", iat: ..., exp: ... }
} catch (err) {
  // TokenExpiredError, JsonWebTokenError
  console.log("Invalid or expired token");
}

// 3. Decode without verification (for reading payload — NOT secure)
const decoded = jwt.decode(token);

// Client stores token in localStorage or httpOnly cookie
// Client sends token in every request:
// Authorization: Bearer <token>
```

**How JWT auth flow works:**
```
1. User logs in → server creates JWT → sends to client
2. Client stores JWT (localStorage / Cookie)
3. Client sends JWT in Authorization header with every request
4. Server verifies JWT on each protected route
5. If valid → allow access; if invalid/expired → 401
```

**JWT is stateless** — the server doesn't store sessions. The token itself contains all needed info.

---

### 31. What is bcrypt? How do you hash passwords?

**bcrypt** is a **password hashing function** designed to be slow and computationally expensive — making brute-force attacks difficult.

**Never store plain passwords in the database!**

```js
// Install: npm install bcrypt
const bcrypt = require("bcrypt");

// HASHING a password (during registration)
async function hashPassword(plainPassword) {
  const saltRounds = 10; // cost factor — higher = slower but more secure
  const hashed = await bcrypt.hash(plainPassword, saltRounds);
  return hashed;
  // "$2b$10$N9qo8uLOickgx2ZMRZoMyeIjZAgcfl7p92ldGxad68LJZdL17lhWy"
}

// COMPARING password during login
async function checkPassword(plainPassword, hashedPassword) {
  const isMatch = await bcrypt.compare(plainPassword, hashedPassword);
  return isMatch; // true or false
}

// In registration route
app.post("/auth/register", async (req, res) => {
  const { email, password } = req.body;

  const hashedPassword = await bcrypt.hash(password, 10);
  const user = await User.create({ email, password: hashedPassword });

  res.status(201).json({ message: "User created!" });
});

// In login route
app.post("/auth/login", async (req, res) => {
  const { email, password } = req.body;
  const user = await User.findOne({ email });

  if (!user || !await bcrypt.compare(password, user.password)) {
    return res.status(401).json({ error: "Invalid credentials" });
  }
  // proceed with token creation...
});
```

---

### 32. What is MongoDB and Mongoose?

**MongoDB** is a **NoSQL document database** that stores data as **JSON-like documents** (BSON) instead of rows/tables. Flexible schema — documents in the same collection can have different fields.

**Mongoose** is an **ODM (Object Document Mapper)** for MongoDB and Node.js — it provides schema validation, relationship management, and a clean API.

```js
// Install: npm install mongoose
const mongoose = require("mongoose");

// 1. Connect to MongoDB
await mongoose.connect(process.env.DB_URL);
console.log("MongoDB connected!");

// 2. Define a Schema
const userSchema = new mongoose.Schema({
  name:      { type: String, required: true, trim: true },
  email:     { type: String, required: true, unique: true, lowercase: true },
  password:  { type: String, required: true, minlength: 6 },
  age:       { type: Number, min: 0, max: 150 },
  role:      { type: String, enum: ["user", "admin"], default: "user" },
  isActive:  { type: Boolean, default: true },
  createdAt: { type: Date, default: Date.now },
});

// 3. Create a Model
const User = mongoose.model("User", userSchema);

// 4. CRUD Operations
// Create
const user = await User.create({ name: "Alice", email: "alice@mail.com", password: "hashed" });
const user2 = new User({ name: "Bob", ... });
await user2.save();

// Read
const all = await User.find();                        // all users
const active = await User.find({ isActive: true });   // filter
const one = await User.findById("64abc...");           // by ID
const byEmail = await User.findOne({ email: "alice@mail.com" });

// Update
await User.findByIdAndUpdate(id, { name: "Alice Updated" }, { new: true });
await User.updateMany({ isActive: false }, { $set: { deletedAt: new Date() } });

// Delete
await User.findByIdAndDelete(id);
await User.deleteMany({ isActive: false });

// Query chaining
const users = await User
  .find({ role: "user" })
  .select("name email -password")  // include/exclude fields
  .sort({ createdAt: -1 })         // sort descending
  .skip(10)                        // pagination
  .limit(10);                      // items per page
```

---

### 33. What is Input Validation? How to validate in Express?

**Input validation** ensures the data sent by clients is correct before processing or saving it. Always validate on the server — never trust client input!

```js
// Install: npm install express-validator
const { body, param, query, validationResult } = require("express-validator");

// Validation middleware chain
const validateRegister = [
  body("name")
    .notEmpty().withMessage("Name is required")
    .isLength({ min: 2, max: 50 }).withMessage("Name must be 2-50 chars"),

  body("email")
    .notEmpty().withMessage("Email is required")
    .isEmail().withMessage("Invalid email format")
    .normalizeEmail(),

  body("password")
    .notEmpty().withMessage("Password is required")
    .isLength({ min: 6 }).withMessage("Password must be at least 6 chars")
    .matches(/\d/).withMessage("Password must contain a number"),

  body("age")
    .optional()
    .isInt({ min: 0, max: 150 }).withMessage("Age must be 0-150"),
];

// Check validation result in route
const handleValidation = (req, res, next) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({ errors: errors.array() });
  }
  next();
};

app.post("/auth/register", validateRegister, handleValidation, async (req, res) => {
  // All inputs are now validated
  const { name, email, password } = req.body;
  // ...
});
```

---

### 34. What is Error Handling in Express?

Proper error handling prevents the server from crashing and returns meaningful error messages.

```js
// 1. Try-catch in async routes
app.get("/users/:id", async (req, res, next) => {
  try {
    const user = await User.findById(req.params.id);
    if (!user) return res.status(404).json({ error: "User not found" });
    res.json({ data: user });
  } catch (err) {
    next(err); // pass to error handler!
  }
});

// 2. Async wrapper utility (avoids try-catch in every route)
const asyncHandler = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};

app.get("/users", asyncHandler(async (req, res) => {
  const users = await User.find();
  res.json({ data: users });
}));

// 3. Global error-handling middleware (MUST have 4 params, MUST be last)
app.use((err, req, res, next) => {
  console.error(err.stack);

  // Mongoose validation error
  if (err.name === "ValidationError") {
    return res.status(400).json({ error: err.message });
  }
  // Mongoose duplicate key
  if (err.code === 11000) {
    return res.status(409).json({ error: "Email already exists" });
  }
  // JWT errors
  if (err.name === "JsonWebTokenError") {
    return res.status(401).json({ error: "Invalid token" });
  }

  // Generic
  res.status(err.status || 500).json({
    error: err.message || "Internal server error"
  });
});

// 4. 404 handler (for undefined routes) — place BEFORE error handler
app.use((req, res) => {
  res.status(404).json({ error: `Route ${req.url} not found` });
});
```

---

### 35. What is Pagination in APIs?

**Pagination** breaks large datasets into smaller pages to improve performance and UX.

```js
// GET /users?page=2&limit=10

app.get("/users", async (req, res) => {
  const page  = parseInt(req.query.page)  || 1;
  const limit = parseInt(req.query.limit) || 10;
  const skip  = (page - 1) * limit;

  const [users, total] = await Promise.all([
    User.find().skip(skip).limit(limit).sort({ createdAt: -1 }),
    User.countDocuments()
  ]);

  res.json({
    data: users,
    pagination: {
      total,
      page,
      limit,
      pages: Math.ceil(total / limit),
      hasNext: page < Math.ceil(total / limit),
      hasPrev: page > 1,
    }
  });
});

// Response:
// {
//   "data": [...10 users...],
//   "pagination": { "total": 95, "page": 2, "limit": 10, "pages": 10 }
// }
```

**Types of pagination:**
- **Offset-based** — `skip` + `limit` (common but slow on large datasets)
- **Cursor-based** — using a cursor/ID as reference (efficient, used in social feeds)

---

### 36. What is the difference between `app.use()` and `app.get()`?

```js
// app.use() — mounts middleware/routers, matches ANY HTTP method
// Can match a path prefix
app.use("/api", apiRouter);        // all methods, path starts with /api
app.use(express.json());            // all routes, all methods
app.use(cors());                    // all routes, all methods

// app.get() — matches ONLY GET requests to EXACT path
app.get("/users", handler);         // only GET /users

// Other method-specific: app.post(), app.put(), app.patch(), app.delete()
app.post("/users", handler);
app.put("/users/:id", handler);

// app.all() — matches ALL methods on that exact path
app.all("/secret", (req, res) => {
  res.send("Whatever method, always respond");
});
```

| | `app.use()` | `app.get()` |
|---|---|---|
| Methods matched | Any | GET only |
| Path matching | Prefix match | Exact match |
| Use for | Middleware, sub-routers | Route handlers |

---

### 37. What is `express.Router()`?

`express.Router()` creates a **mini Express app** for modular route handling. Recommended for organizing routes in large applications.

```js
// routes/users.js
const express = require("express");
const router = express.Router();
const { authenticate, authorize } = require("../middleware/auth");
const { validateUser } = require("../middleware/validate");

// All routes here are relative to where the router is mounted
router.get("/",       authenticate, getUsers);
router.get("/:id",    authenticate, getUserById);
router.post("/",      authenticate, authorize("admin"), validateUser, createUser);
router.patch("/:id",  authenticate, updateUser);
router.delete("/:id", authenticate, authorize("admin"), deleteUser);

module.exports = router;

// ────────────────────────────────────
// routes/index.js
const express = require("express");
const router = express.Router();
const userRoutes = require("./users");
const postRoutes = require("./posts");
const authRoutes = require("./auth");

router.use("/auth",  authRoutes);
router.use("/users", userRoutes);
router.use("/posts", postRoutes);

module.exports = router;

// ────────────────────────────────────
// app.js
const routes = require("./routes");
app.use("/api/v1", routes);

// Final URLs:
// POST /api/v1/auth/login
// GET  /api/v1/users
// GET  /api/v1/users/42
```

---

### 38. What is a RESTful API Best Practices Checklist?

```
✅ Use nouns for endpoints: /users, /products (not /getUsers)
✅ Use correct HTTP methods: GET=read, POST=create, PATCH=update, DELETE=remove
✅ Return correct status codes: 200, 201, 400, 401, 403, 404, 500
✅ Version your API: /api/v1/
✅ Handle errors consistently with a standard response format
✅ Validate all input (express-validator or Joi)
✅ Paginate list responses: ?page=1&limit=10
✅ Use HTTPS in production
✅ Add CORS headers
✅ Never expose passwords or sensitive data in responses
✅ Use environment variables for secrets (.env)
✅ Authenticate with JWT / session tokens
✅ Log requests (morgan or winston)
✅ Rate limit to prevent abuse (express-rate-limit)
✅ Use meaningful error messages
```

**Consistent response format:**
```json
// Success
{
  "status": "success",
  "message": "User created",
  "data": { "id": 1, "name": "Alice" }
}

// Error
{
  "status": "error",
  "message": "Email is required",
  "errors": [{ "field": "email", "msg": "Email is required" }]
}
```

---

### 39. What is Rate Limiting?

**Rate limiting** restricts how many requests a client can make in a given time window — protects APIs from abuse, DoS attacks, and excessive usage.

```js
// Install: npm install express-rate-limit
const rateLimit = require("express-rate-limit");

// General rate limiter
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,  // 15 minutes
  max: 100,                   // max 100 requests per window
  message: { error: "Too many requests, please try again later" },
  standardHeaders: true,      // Return rate limit info in headers
  legacyHeaders: false,
});

app.use(limiter); // apply to all routes

// Stricter limiter for auth routes
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,          // only 5 login attempts per 15 minutes
  message: { error: "Too many login attempts, try again in 15 minutes" },
});

app.post("/auth/login", authLimiter, loginHandler);
```

---

### 40. What is Async/Await in Node.js and Express?

Async/await makes asynchronous code look synchronous and is the **modern standard** for handling async operations in Node.js.

```js
// Problem with callbacks (callback hell)
app.get("/dashboard", (req, res) => {
  getUserById(req.user.id, (err, user) => {
    if (err) return res.status(500).json({ error: err });
    getOrdersByUser(user.id, (err, orders) => {
      if (err) return res.status(500).json({ error: err });
      getRecentActivity(user.id, (err, activity) => {
        if (err) return res.status(500).json({ error: err });
        res.json({ user, orders, activity }); // deeply nested!
      });
    });
  });
});

// Clean async/await version ✅
app.get("/dashboard", async (req, res) => {
  try {
    const user     = await getUserById(req.user.id);
    const orders   = await getOrdersByUser(user.id);
    const activity = await getRecentActivity(user.id);

    res.json({ user, orders, activity });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// Run async operations in parallel (faster!)
app.get("/dashboard", async (req, res) => {
  try {
    const [orders, activity] = await Promise.all([
      getOrdersByUser(req.user.id),
      getRecentActivity(req.user.id),
    ]);
    res.json({ orders, activity });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});
```

---

### 41. What is the `http` Module in Node.js?

The built-in **`http`** module lets you create a simple HTTP server without any framework.

```js
const http = require("http");

// Create a basic server
const server = http.createServer((req, res) => {
  // req = IncomingMessage, res = ServerResponse
  const { method, url } = req;

  res.writeHead(200, { "Content-Type": "application/json" });

  if (method === "GET" && url === "/") {
    res.end(JSON.stringify({ message: "Hello World!" }));
  } else if (method === "GET" && url === "/users") {
    res.end(JSON.stringify({ users: ["Alice", "Bob"] }));
  } else {
    res.writeHead(404);
    res.end(JSON.stringify({ error: "Not found" }));
  }
});

server.listen(3000, () => {
  console.log("Server running on http://localhost:3000");
});

// Express is built on top of http module
// The above = what Express does internally (but much more feature-rich)
```

---

### 42. What is Streaming in Node.js?

**Streams** are objects that let you read or write data **piece by piece** instead of loading everything into memory at once. Ideal for large files, video, or real-time data.

```js
const fs = require("fs");

// 4 types of streams:
// Readable — you read from it (fs.createReadStream)
// Writable — you write to it (fs.createWriteStream)
// Duplex  — read and write (net.Socket)
// Transform — modify data as it passes through (zlib compression)

// Reading a large file with streams (memory efficient)
const readable = fs.createReadStream("bigfile.txt", { encoding: "utf8" });
readable.on("data",  chunk => process.stdout.write(chunk));
readable.on("end",   ()    => console.log("Done reading!"));
readable.on("error", err   => console.error(err));

// Writing with a stream
const writable = fs.createWriteStream("output.txt");
writable.write("First chunk\n");
writable.write("Second chunk\n");
writable.end(); // close the stream

// Piping — connect readable → writable (most efficient)
const readStream  = fs.createReadStream("source.txt");
const writeStream = fs.createWriteStream("dest.txt");
readStream.pipe(writeStream); // copy file via streaming

// In Express — stream a file response
app.get("/download", (req, res) => {
  res.setHeader("Content-Disposition", "attachment; filename=report.pdf");
  const fileStream = fs.createReadStream("./reports/report.pdf");
  fileStream.pipe(res); // stream directly to client
});
```

---

### 43. What is `Promise.all()` in the context of Node.js?

In Node.js backend, `Promise.all()` is frequently used to run **multiple async DB queries or API calls in parallel**, reducing total response time.

```js
// Sequential — slow (each waits for previous)
const user    = await User.findById(id);      // 100ms
const orders  = await Order.find({ userId: id });  // 200ms
const reviews = await Review.find({ userId: id }); // 150ms
// Total: 450ms

// Parallel with Promise.all — fast (all run concurrently)
const [user, orders, reviews] = await Promise.all([
  User.findById(id),                // 100ms
  Order.find({ userId: id }),       // 200ms  \
  Review.find({ userId: id }),      // 150ms   ├── all at once!
]);
// Total: ~200ms (longest one)

// With error handling
try {
  const [userCount, orderCount] = await Promise.all([
    User.countDocuments(),
    Order.countDocuments(),
  ]);
  res.json({ users: userCount, orders: orderCount });
} catch (err) {
  // If ANY promise rejects, the whole thing rejects
  next(err);
}
```

---

### 44. What is the difference between SQL and NoSQL databases?

| Feature | SQL (PostgreSQL, MySQL) | NoSQL (MongoDB) |
|---|---|---|
| Data format | Tables with rows & columns | Documents (JSON-like), key-value, graphs |
| Schema | Fixed, rigid schema | Flexible, dynamic schema |
| Relationships | Joins between tables | Embedding or referencing |
| Scaling | Vertical (bigger server) | Horizontal (more servers) |
| ACID | ✅ Strong ACID compliance | ✅ (MongoDB supports ACID since v4) |
| Best for | Complex queries, relationships | Hierarchical data, fast iteration |
| Examples | PostgreSQL, MySQL, SQLite | MongoDB, Redis, Cassandra, DynamoDB |

```js
// SQL (with pg / sequelize)
SELECT * FROM users WHERE email = 'alice@mail.com';

// MongoDB (with Mongoose)
User.findOne({ email: "alice@mail.com" });
```

**When to use MongoDB:**
- Flexible schemas (evolving data models)
- Hierarchical/nested data
- High write loads
- Real-time applications

**When to use SQL:**
- Complex relationships between data
- Strong consistency required
- Financial/transactional data
- Complex reporting queries

---

### 45. How do you structure a Node.js/Express project?

A well-organized project structure makes code maintainable and scalable.

```
my-project/
├── src/
│   ├── config/         ← DB, app config
│   │   ├── database.js
│   │   └── app.js
│   ├── middleware/     ← auth, error handler, validation
│   │   ├── auth.js
│   │   ├── validate.js
│   │   └── errorHandler.js
│   ├── models/         ← Mongoose schemas
│   │   ├── User.js
│   │   └── Post.js
│   ├── controllers/    ← business logic (request handlers)
│   │   ├── userController.js
│   │   └── postController.js
│   ├── services/       ← reusable business logic (optional)
│   │   └── emailService.js
│   ├── routes/         ← URL → controller mapping
│   │   ├── index.js
│   │   ├── users.js
│   │   └── posts.js
│   └── utils/          ← helper functions
│       └── helpers.js
├── .env                ← environment variables (NEVER commit!)
├── .gitignore
├── package.json
└── index.js            ← entry point (start server)
```

**MVC-inspired pattern:**
```js
// Route → Controller → Model pattern
// routes/users.js
router.get("/:id", userController.getUser);

// controllers/userController.js
exports.getUser = async (req, res, next) => {
  const user = await User.findById(req.params.id);
  res.json({ data: user });
};

// models/User.js
const User = mongoose.model("User", userSchema);
```

---

*End of Node.js & APIs Reference*

---

### 46. What is the difference between `npm install` and `npm ci`?

-   **`npm install`:** Installs dependencies from `package.json`. It may update the `package-lock.json` if it finds newer compatible versions. Used during development.
-   **`npm ci` (Clean Install):** It is faster and more strict. It **requires** a `package-lock.json` and will only install what is exactly in that file. It deletes the `node_modules` folder before starting. Used in **CI/CD pipelines** or production environments to ensure consistent builds.

---

### 47. How do you handle Environment Variables in Node.js?

Environment variables are used to store sensitive data (like API keys, DB passwords) outside the code.

**How to use:**
1.  Create a `.env` file: `PORT=3000`.
2.  Install `dotenv`: `npm install dotenv`.
3.  Load it at the top of your entry file:
    ```js
    require('dotenv').config();
    const port = process.env.PORT || 5000;
    ```

**Important:** Never commit `.env` files to Git.

---

### 48. What is the purpose of `node_modules` and should it be committed to Git?

-   **Purpose:** It is the folder where npm/yarn stores all the downloaded packages and their dependencies required for your project.
-   **Should it be committed?** **NO.** It is usually very large and can be reconstructed anytime using `npm install`.
-   **Best Practice:** Add `node_modules` to your `.gitignore` file.

---

### 49. What are `dependencies`, `devDependencies`, and `peerDependencies`?

-   **`dependencies`:** Packages required for the application to **run** in production (e.g., `express`, `mongoose`).
-   **`devDependencies`:** Packages only needed for **development and testing** (e.g., `nodemon`, `jest`, `eslint`).
-   **`peerDependencies`:** Packages that your package expects the **host** project to already have installed (usually for plugins or libraries).

---

### 50. What is the "Thread Pool" in Node.js? (Libuv)

While the JavaScript engine (V8) is single-threaded, Node.js uses a library called **Libuv** to handle heavy operations (like File System, DNS, Crypto) in a **Thread Pool**.

**Key points:**
-   By default, the thread pool size is **4**.
-   It allows Node.js to perform multi-threaded work in the background without blocking the main event loop.
-   You can increase it using `process.env.UV_THREADPOOL_SIZE`.

---

*End of Node.js & APIs Reference*
