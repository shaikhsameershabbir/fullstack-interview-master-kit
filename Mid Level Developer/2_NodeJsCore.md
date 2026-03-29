# Node.js & Express – Mid-Level Developer Interview Reference

---

## 🟢 Section 1: Node.js Core Architecture & Internals

---

### 1. Explain Node.js Architecture in-depth.

**Definition:** Node.js is an asynchronous, event-driven JavaScript runtime built on Chrome's V8 engine. Its core architecture is based on a **Single-Threaded Event Loop** model, but "Single-Threaded" is a bit of a misnomer when looking at the whole system.

**Internal Mechanics:**
1.  **V8 Engine:** Developed by Google, it compiles JS to machine code. It handles memory allocation (Heap) and garbage collection.
2.  **Libuv (The Backbone):** A C library that provides the **Event Loop** and the **Thread Pool**. While JS runs in one thread, libuv offloads blocking operations.
3.  **Thread Pool (Worker Pool):** By default, libuv has a pool of **4 threads** (configurable via `UV_THREADPOOL_SIZE`). It handles disk I/O, DNS lookups, and crypto functions that don't have an asynchronous system API.
4.  **OS Kernel:** For network I/O (sockets), libuv doesn't use threads. It uses non-blocking system calls (like `epoll` on Linux, `kqueue` on macOS) provided by the OS kernel. This is how Node handles thousands of concurrent connections effortlessly.

**Architectural Impact:**
Node.js is optimized for **I/O-bound** applications (web servers, real-time chats). Because it doesn't spawn a new thread for every request (like Java or PHP), it consumes very little memory. However, it is fundamentally weak for **CPU-bound** tasks because a long-running calculation on the main thread will "freeze" the entire server.

---

### 2. What is Libuv and how does it interface with the OS?

**Definition:** **Libuv** is a multi-platform support library with a focus on asynchronous I/O. It was originally developed for Node.js but is now used by other projects like Julia and Luvit.

**How it interfaces with the OS:**
-   **Network I/O:** Libuv uses an "Event Demultiplexer". When you're waiting for data on a socket, libuv registers the file descriptor with the OS kernel (using `epoll`). The OS notifies libuv when data is ready. This is "non-blocking" because the thread doesn't wait; it just moves on to other tasks.
-   **File I/O:** Most OS kernels do not provide a truly asynchronous API for file systems. Therefore, libuv uses its **Thread Pool** to simulate asynchrony. A thread from the pool blocks on the file read, and when it's done, it signals libuv to put the callback in the event loop.

---

### 3. Deep Dive: Node.js Event Loop Phases

The event loop is a continuous cycle that manages the execution of callbacks. It consists of several distinct phases:

1.  **Timers Phase:** Executes callbacks scheduled by `setTimeout()` and `setInterval()`. Note: The delay is a *minimum* wait time, not an exact one.
2.  **Pending Callbacks Phase:** Executes I/O callbacks deferred from the previous loop iteration (e.g., some types of TCP errors).
3.  **Idle, Prepare Phase:** Internal phases used by Node for housekeeping.
4.  **Poll Phase (The Heart):** 
    -   Calculates how long it should block and poll for I/O.
    -   Processes events in the poll queue.
    -   If the queue is empty and there are `setImmediate` scripts, the loop ends the Poll phase and moves to the Check phase.
5.  **Check Phase:** Executes `setImmediate()` callbacks.
6.  **Close Callbacks Phase:** Executes callbacks for closed connections, like `socket.on('close', ...)`.

**Critical Knowledge: Microtasks (`process.nextTick` & Promises)**
Microtasks are **not** part of the libuv phases. They are handled by the V8 engine and are executed **immediately after the current operation finishes**, regardless of which phase the event loop is in. `process.nextTick` has priority over Promise `.then()` callbacks.

---

### 4. Worker Threads vs Cluster Module vs Child Processes

| Feature | Cluster Module | Worker Threads | Child Processes |
|---|---|---|---|
| **Mechanism** | Multi-process (Master/Worker). | Multi-thread (Main/Worker). | Spawning external processes. |
| **Memory** | Shared nothing (IPC messages). | **Shared Memory** (`SharedArrayBuffer`). | Shared nothing (IPC/Streams). |
| **Port Sharing**| Yes (all workers share port 80). | No (Main handles networking). | No. |
| **Use Case** | **Vertical Scaling** of API servers. | **Heavy Computation** (JS logic). | Running Shell scripts/Python. |

**Real-world Scenario:**
Use **Cluster** to maximize your server's CPU cores for a web server. Use **Worker Threads** if you need to perform data compression or image processing in response to a request without blocking your API.

---

### 5. What are Streams and the "Backpressure" Problem?

**Definition:** Streams are collections of data that might not be available all at once and don't fit in memory.

**Types:** `Readable`, `Writable`, `Duplex` (TCP Sockets), and `Transform` (Zlib, Crypto).

**The Backpressure Problem:**
Imagine you are reading a 5GB file (Readable) and writing it to a slow network connection (Writable). The reader is much faster than the writer. If you don't control the flow, the unwritten data will buffer in memory, eventually causing an "Out of Memory" (OOM) crash.

**The Solution:**
Modern streams use `.pipe()`. Internally, if the writer's buffer is full, it signals the reader to stop. When the writer clears its buffer, it emits a `'drain'` event, and the reader resumes. This is automatic flow control.

---

### 6. Buffer Internals: How Node.js handles memory outside V8

**Definition:** A **Buffer** is a subclass of the `Uint8Array` type and is used to represent a fixed-length sequence of bytes. 

**Internals:**
V8 has a memory limit (usually 1.5GB to 4GB). Buffers are allocated **outside the V8 heap** (in the system memory). This is why you can handle large buffers in Node without triggering constant V8 Garbage Collection cycles.

**Why use them?**
JS strings are UTF-16 and expensive to manipulate for binary data. Buffers allow you to work with raw binary packets (TCP, File, PNG) efficiently.

---

### 7. How to manage "Memory Leaks" in Node.js Production?

**Detection Tools:**
1.  **`node --inspect`**: Attaches Chrome DevTools.
2.  **`heapdump`**: A module to take a snapshot of the memory.
3.  **`clinic.js`**: A powerful diagnostic tool for Node.

**Common Culprits:**
-   **Global Variables:** Never collected.
-   **Closures:** Accidentally holding references to large objects in the outer scope.
-   **Event Listeners:** Attaching `emitter.on('data', ...)` and never calling `.removeListener()`, especially in a loop or connection handler.
-   **Caching:** Using a simple `{}` as a cache without an expiration (TTL) or maximum size.

**Best Practice:** Always use a **WeakMap** for caching objects when you want the object to be garbage-collected automatically if it's no longer used elsewhere.

---

## 🚂 Section 2: Express.js & Advanced API Patterns

---

### 8. The Middleware Pattern in Express

**Mechanism:** Express is essentially a series of middleware function calls. Each function has the `req`, `res`, and `next()`. 

**The "next()" Trap:**
Crucially, if you don't call `next()` and don't send a response, the request will hang forever. If you call `next()` *after* sending a response, you might trigger "Headers already sent" errors if the next middleware tries to set a status code.

**Async Middleware:**
Express 4 doesn't catch errors in `async` middleware automatically. You **must** wrap them in a try/catch and call `next(err)`. Express 5 (currently in beta) fixes this.

---

### 9. Custom Error Handling Architecture

A mid-level developer shouldn't just `console.log(err)`. You need a centralized error handler.

**Implementation Pattern:**
1.  Create a custom `ApiError` class that extends `Error` and includes a `statusCode`.
2.  Use a wrapper function for controllers to catch async errors.
3.  Define a global error middleware at the bottom of the stack.

```js
// Global Error Handler
app.use((err, req, res, next) => {
  const status = err.statusCode || 500;
  // Use a structured logger like Pino instead of console
  logger.error({ 
    message: err.message, 
    stack: err.stack,
    requestId: req.headers['x-request-id'] 
  });
  
  res.status(status).json({
    success: false,
    error: err.name,
    message: err.message
  });
});
```

---

### 10. API Security Best Practices (Beyond the basics)

1.  **JWT Secret Rotation:** Don't use a static secret forever.
2.  **Rate Limiting with Redis:** Use `rate-limiter-flexible` with a Redis backend so the limit is shared across all processes in your cluster.
3.  **NoSQL Injection:** Even with MongoDB, you should use `mongo-sanitize` to prevent attackers from sending objects with `$gt` instead of strings.
4.  **Security Headers:** `helmet` is the bare minimum. Specifically, look at **HSTS** (Strict-Transport-Security) and **CSP** (Content Security Policy).
5.  **Payload Limits:** Use `express.json({ limit: '10kb' })` to prevent "Entity Too Large" attacks that crash the server by sending massive JSON payloads.

---

### 11. What is a "Correlation ID" and why is it essential?

**Problem:** In a system with many logs, how do you see all logs related to a specific user's request?

**Solution:**
-   Generate a UUID at the `API Gateway` or the first middleware.
-   Attach it to the request: `req.id = uuid()`.
-   Include it in every log statement.
-   Forward it to downstream microservices in the headers.

**Architectural Impact:** This turns a "sea of logs" into a traceable flow, allowing you to debug exactly what happened during a specific failed request.

---

### 12. Graceful Shutdown: The SIGTERM Pattern

When running in a container (Docker/K8s), the orchestrator sends a `SIGTERM` signal to tell your app to stop.

**Why Graceful?**
If you just "kill" the process, you might interrupt a SQL transaction half-way or cut off a user in the middle of a file download.

**Pattern:**
1.  Listen for `process.on('SIGTERM')`.
2.  Stop the server from accepting new connections (`server.close()`).
3.  Wait for active requests to finish (Express does this automatically with `server.close`).
4.  Close DB connections cleanly.
5.  Perform `process.exit(0)`.

---

### 13. How to handle "Large File Uploads" (Direct vs Multi-part)?

**Standard way (Multer + Storage):**
-   File comes to Express.
-   Multer saves it to a temp folder on your server.
-   You then upload it to S3.
-   **Problem:** Your server is doing double the work and using its own disk space!

**Mid-level way (S3 Pre-signed URLs):**
-   Client asks API: "I want to upload a file".
-   API asks S3: "Give me a temporary upload link".
-   API returns the link to the client.
-   Client uploads **directly** to S3.
-   **Benefit:** Zero load on your Node.js server. Your server only handles small JSON messages.

---

### 14. Performance Optimization in Express

1.  **Gzip Compression:** Use the `compression` middleware to shrink response sizes.
2.  **Fast JSON Stringification:** Use `fast-json-stringify` if you have high-traffic JSON responses.
3.  **Avoid `Array.includes` in Loops:** On large datasets, use a **Set** or **Map** for O(1) lookups.
4.  **Offload CPU work:** Send heavy tasks (email, image processing) to a background worker via a **Message Queue** (RabbitMQ/SQS).

---

### 15. The "Circular Dependency" Problem in Large Projects

**Problem:** Module A requires Module B, and Module B requires Module A. This often happens between "Users" and "Teams" controllers/services.

**Impact:** Node.js handles this by returning an **incomplete/empty object** for the second module in the cycle, which leads to `TypeError: undefined is not a function`.

**How to avoid:**
1.  **Refactor:** Extract shared logic into a third "Service C".
2.  **Dependency Injection:** Pass the dependency as an argument to a function or constructor instead of `require`-ing it at the top level.
3.  **Move the requires:** (Last resort) Move the `require` call inside the specific function that uses it.

---

### 16. How does `require` actually work? (The Module Wrapper)

**Internal Mechanics:**
When you call `require('./module')`, Node.js doesn't just execute the file. It performs these steps:
1.  **Resolving:** Findings the absolute path of the file.
2.  **Loading:** Reading the file content.
3.  **Wrapping:** Node wraps your code in a function wrapper:
    ```js
    (function(exports, require, module, __filename, __dirname) {
      // Your code lives here
    });
    ```
4.  **Evaluating:** The V8 engine executes the wrapped function.
5.  **Caching:** The `module.exports` object is cached. Subsequent `require` calls for the same file return the cached object instantly.

**Why the Wrapper?**
It provides "local" variables like `__dirname` and `module` that look global but are actually function arguments. It also keeps your variables from polluting the real global scope.

---

### 17. What is `AsyncLocalStorage` and why use it?

**Problem:** In Node.js, since it's single-threaded and async, you lose "Thread Local Storage". If you want to track a `requestId` across 20 different function calls without passing it as an argument everywhere, it's very difficult.

**Solution:** `AsyncLocalStorage` (from `node:async_hooks`) allows you to store data that is "local" to an asynchronous execution flow.
-   **Use Case:** Tracing logs. You store the `traceId` at the start of the request, and every log function down the line can retrieve it automatically from the store.

---

### 18. `process.nextTick` vs `setImmediate`: The "Starvation" Issue

-   **`process.nextTick`**: Adds the callback to the "Next Tick Queue". This queue is processed **immediately** after the current operation, before the event loop continues to the next phase.
-   **`setImmediate`**: Adds the callback to the "Check" phase of the event loop.

**The Danger (Starvation):**
If you recursively call `process.nextTick`, the event loop will **never** move to the next phase (I/O, Timers, etc.). This "starves" the event loop, making your server unresponsive to new requests. `setImmediate` is safer because it allows other phases to run first.

---

### 19. Handling "Large Data": V8 Heap Limits

**Problem:** By default, Node.js might limit the heap memory to ~1.5GB. If you try to process a 3GB JSON object, the process will crash with `FATAL ERROR: Ineffective mark-compacts near heap limit`.

**Solutions:**
1.  **Limit increase:** Use `--max-old-space-size=4096` to increase the limit to 4GB.
2.  **Streams:** (Better) Don't load the data. Process it chunk by chunk.
3.  **Spawn:** Offload to a child process that has its own memory space.

---

### 20. Security: ReDoS (Regular Expression Denial of Service)

**Problem:** Certain "Evil RegEx" patterns can take exponential time to process if given a specific input (Backtracking). 
-   Example: `/(a+)+$/` on a string of 100 "a"s followed by an "x".

**Impact:** Since Node is single-threaded, one "Evil RegEx" will block the entire event loop for seconds or even minutes, effectively DDOSing your own server.

**Prevention:** Use libraries like `safe-regex` to check your patterns or avoid complex nested quantifiers in user-supplied input.

---

### 21. Event Emitter: Memory Leak Prevention

**Problem:** If you keep adding listeners to a global emitter (like `process.on('message')`) and never remove them, your memory usage will grow forever.

**Best Practices:**
1.  **`once()`**: Use this if you only need the event to fire once. Node automatically removes the listener after execution.
2.  **`removeListener()`**: Always clean up in your `destroy` or `unmount` logic.
3.  **`setMaxListeners(n)`**: Node warns you if you add more than 10 listeners. Don't just silence the warning; investigate why you have so many.

---

### 22. Path: `join()` vs `resolve()`

-   **`path.join('/a', '/b', 'c')`**: Just joins the segments using the platform-specific separator. Result: `\a\b\c` (on Windows).
-   **`path.resolve('/a', '/b', 'c')`**: Processes the segments like a sequence of `cd` commands. It always returns an **absolute path** based on the current working directory.

**Rule of Thumb:** Use `resolve` when you need an absolute path for file operations. Use `join` for simple path string manipulation.

---

### 23. `URL` Object vs `url.parse` (Legacy)

**Decision:** Always use the `new URL()` constructor (WHATWG standard).
-   **Why?** `url.parse` is legacy and has security vulnerabilities regarding how it parses certain characters. The WHATWG `URL` object is more secure, follows the same standard as browsers, and handles encoding much better.

---

### 24. Proper Stream Handling: `pipeline` vs `.pipe()`

**Problem:** `source.pipe(dest)` does **not** automatically destroy the streams if an error occurs. This can lead to memory leaks (open file descriptors).

**Solution:** Use `stream.pipeline` (or `stream/promises` version).
```js
const { pipeline } = require('node:stream/promises');

await pipeline(
  fs.createReadStream('file.txt'),
  zlib.createGzip(),
  fs.createWriteStream('file.txt.gz')
);
// ✅ Automatically handles errors and closes ALL streams safely.
```

---

### 25. Worker Threads: `MessageChannel` and `MessagePort`

**Definition:** Communication between the main thread and workers happens via "Message Passing". 

**Internals:**
A `MessageChannel` consists of two `MessagePort`s. You can pass one port to a worker. This allows for direct, 2-way asynchronous communication between workers without going through the main thread (improving performance in complex multi-threaded setups).

---

### 26. Prototype Pollution Protection

**Problem:** If an attacker sends a JSON payload like `{"__proto__": {"isAdmin": true}}`, a naive object merger (`Object.assign` or some versions of `lodash.merge`) might modify the base `Object.prototype`, making **every** object in your app have `isAdmin: true`.

**Prevention:** 
1.  Freeze the prototype: `Object.freeze(Object.prototype)`.
2.  Use `Map` instead of plain objects for dynamic keys.
3.  Use `Object.create(null)` for objects that don't need a prototype.

---

### 27. Why "Double Equals" (==) is a danger in Node.js?

While this is basic JS, in Node.js backend logic (like Auth check), it's fatal.
`if (user.id == req.body.id)`
If `user.id` is `123` (Number) and `req.body.id` is `"123"` (String), it passes. But if `req.body.id` is an array `[123]`, it **also passes**. Always use `===`.

---

### 28. "Zero-Downtime" Deployment: PM2 Reload vs Restart

-   **`pm2 restart`**: Kills the process and starts a new one. There is a "gap" where requests are dropped.
-   **`pm2 reload`**: Starts a new process first, waits for it to be ready, and then kills the old one. Zero packets are lost. 

**Mid-Level Tip:** For production APIs, `reload` is the only acceptable way to deploy without affecting users.

---

### 29. CPU Profiling: How to find the "Blocker"?

If your API is slow, you can use **`clinic.js flame`**.
-   It generates a "Flamegraph".
-   The **Horizontal Axis** shows the function name.
-   The **Width** shows how much time was spent in that function.
-   **Red/Hot** areas indicate the bottleneck.

---

### 30. Internal Binding vs Addons

-   **Internal Binding:** How Node's JS code accesses the C++ built-in modules (like `fs` or `crypto`).
-   **C++ Addons:** Custom C++ code you write yourself to extend Node's capabilities. Use **N-API** (Node-API) to ensure your addon works across different Node.js versions without recompiling.

---

### 31. Handling "Unhandled Rejections" and "Uncaught Exceptions"

**Problem:** If an async function throws an error and isn't wrapped in a `try/catch`, it becomes an "Unhandled Rejection". If a synchronous error isn't caught, it's an "Uncaught Exception".

**Mid-Level Best Practice:**
1.  **Don't let the app stay up:** Use `process.on('uncaughtException')` to log the error and then **exit** (`process.exit(1)`). An unhandled exception means the app is in an "Unpredictable State" (e.g., a database connection might be hung).
2.  **Orchestrator handles restart:** Let Docker or PM2 restart the process.
3.  **Trace:** Use `process.on('unhandledRejection')` to catch async errors that were missed.

---

### 32. `Buffer.alloc` vs `Buffer.allocUnsafe`

-   **`Buffer.alloc(size)`**: Initializes the buffer with zeros.
    -   *Pros:* Secure. No risk of reading old data.
    -   *Cons:* Slightly slower.
-   **`Buffer.allocUnsafe(size)`**: Does NOT initialize the buffer. It gives you a block of raw memory as-is.
    -   *The Risk:* It might contain **sensitive data** (passwords, JWTs) from a previous operation. Only use if performance is critical and you are immediately overwriting the entire buffer.

---

### 33. `stream.Readable.from()` and Async Iterators

**Definition:** Introduced in Node 12, this allows you to turn any **Async Iterable** (like a generator or a DB cursor) into a Node.js Stream.
-   **Benefit:** You can use the `for await...of` syntax, which is much more readable than the `'data'` event listener.

---

### 34. How `util.promisify` works under the hood?

**Mechanism:** It takes a function with the standard `(err, value)` callback pattern and returns a function that returns a **Promise**.
-   **Mid-Level Nuance:** It uses a special symbol `util.promisify.custom` to allow developers to provide a manual promise implementation for functions that don't follow the standard callback pattern.

---

### 35. The `Diagnostics Channel` (node:diagnostics_channel)

**Definition:** A stable API for creating and subscribing to "Telemtery Channels". 
-   **Use Case:** An APM (Application Performance Monitoring) tool like New Relic can "subscribe" to the `http` channel to automatically time every incoming request without you having to add tracing code to every controller.

---

### 36. Security: Command Injection via `child_process.exec`

**Problem:** Using `exec("rm -rf " + userInput)` allows an attacker to send `; sudo rm -rf /`.
-   **Prevention:** Always use **`child_process.spawn`** or **`execFile`**. They take the command and the arguments as **separate** items, meaning the OS treats the user input as a literal string, not as a part of the command itself.

---

### 37. Middleware: How `body-parser` works?

**Internal:** Express doesn't "have" the body by default. `body-parser` sits on the `req` (which is a Readable Stream). It listens for the `'data'` events, buffers the chunks into a string, and then calls `JSON.parse(string)` before attaching it to `req.body`.

---

### 38. Performance: Object Pooling in Node.js

**Problem:** Rapidly creating and destroying thousands of small objects (like DB result wrappers) puts massive pressure on the **V8 Garbage Collector**, leading to "GC pauses" that slow down your API.
-   **Solution (Object Pool):** You create a pool of objects and "Rent" them. When you are done, you "Return" them to the pool instead of letting them be destroyed. This keeps memory usage stable and the GC idle.

---

### 39. `perf_hooks` for Precise Measurement

Don't use `Date.now()`. It's only accurate to 1ms.
-   **Solution:** Use `performance.now()`. It provides "High Resolution" timestamps accurate to **microseconds**. Essential for measuring the latency of a critical encryption function or a cache lookup.

---

### 40. "Module Preload" (--require)

**Definition:** Using the `--require` or `-r` flag when starting node. 
-   **Use Case:** Starting a monitoring tool or a `.env` loader (`node -r dotenv/config app.js`). This ensures that the configuration is loaded **before** any other line of your application code runs.

---

### 41. Dealing with Timezones (process.env.TZ)

**Mid-Level Best Practice:** Always run your Node.js servers in **UTC**.
-   Set `TZ=UTC` in your environment variables. 
-   Save all dates to the database as ISO strings (`toJSON()`). 
-   Only convert to local time on the **Frontend** based on the user's browser settings.

---

### 42. Node.js `node:test` runner vs Jest

-   **`node:test`**: Built-in (as of Node 18). Extremely fast because it has zero depedencies. Best for library authors.
-   **Jest**: Feature-rich (mocking, snapshots, coverage).
-   **Cons of Jest in Node:** Jest "fakes" the module environment to provide its features, which can lead to bugs where code works in tests but fails in the real Node environment. `node:test` runs in the real environment.

---

### 43. Using `AbortController` for Timeouts

**Problem:** The built-in `fetch` in Node.js doesn't have a "timeout" option.
**Solution:**
```js
const controller = new AbortController();
const timeout = setTimeout(() => controller.abort(), 5000);

try {
  const res = await fetch(url, { signal: controller.signal });
} finally {
  clearTimeout(timeout);
}
```

---

### 44. Subprocess stdio: `pipe` vs `inherit` vs `ignore`

-   **`pipe`**: The child's output is captured in a stream you can read from in JS.
-   **`inherit`**: The child's output goes directly to your terminal.
-   **`ignore`**: The output is discarded. (Best for "Fire and forget" background commands).

---

### 45. The `WHATWG` URL API vs Legacy `url` module

-   **Legacy:** `url.parse('...')`. Has security issues (Open Redirects) and is being deprecated.
-   **WHATWG:** `new URL('...')`. Standardized across Browsers and Node. It handles non-standard characters securely and is the only API you should use.

---

*Expansion Complete (45 Questions)*
