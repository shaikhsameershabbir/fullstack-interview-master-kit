## 10) Non-blocking I/O (why Node is fast)

### Definition (technical)
**Non-blocking I/O** is an execution model where initiating I/O does not block the JavaScript thread; instead, the runtime/OS performs or waits for the operation asynchronously and later schedules a callback or promise continuation on the event loop when the result is ready (often via evented OS APIs or libuv’s thread pool).

“Node is fast” isn’t a magical claim. It’s a specific design choice:

> Don’t waste the JavaScript thread waiting on slow things (disk/network). Start the work, then come back later when it’s ready.

If you truly understand non-blocking I/O, you’ll understand:
- why Node is great for APIs and real-time services
- why sync filesystem calls in a request handler are a disaster
- why “async” can still become slow (thread pool saturation)

---

### Basic intuition (simple)
Blocking I/O feels like:
- “start I/O”
- **wait**
- “continue”

Non-blocking I/O feels like:
- “start I/O”
- “when it’s done, call me / resolve a promise”
- “continue doing other work now”

Node’s main thread is like a receptionist:
- don’t sit on hold with one customer
- take the next request and handle completed work as it returns

---

### Internal working (engine level)
#### Two broad async strategies in Node
Node uses two broad strategies depending on the type of I/O:

1) **OS-backed evented I/O**
- Networking sockets are the classic case.
- The OS can notify when data is readable/writable.

2) **Thread pool offloading (libuv)**
- For some operations (commonly filesystem, some DNS paths, some crypto), Node uses a thread pool.
- Work happens off the JS thread.
- Completion is sent back to the event loop, then your callback runs on the main thread.

In both cases, the important point is:
- JS callbacks always run on the main thread, **later**, when scheduled by the event loop.

#### “Why Node is fast” (precise version)
Node is fast for **I/O-bound** workloads because:
- the event loop multiplexes lots of connections
- Node avoids a thread-per-connection model

Node is not automatically fast for **CPU-bound** workloads:
- CPU-heavy JS blocks the call stack → everything gets delayed

##### Checkpoint
Say it clearly:
- What kind of workload is Node best at, and what kind is it worst at?

---

### Edge cases (where people get surprised)
#### 1) Sync APIs destroy concurrency
If you use sync I/O in a request handler:
- the event loop is blocked
- other requests wait behind it
- latency spikes

Examples:
- `fs.readFileSync`
- `crypto.pbkdf2Sync`

#### 2) “Async” can still be slow (thread pool saturation)
If you start too many thread-pool-backed operations at once:
- they queue up
- completion callbacks arrive late
- throughput flattens under load

Symptom pattern:
- you aren’t blocking the event loop with sync code
- but requests still get slower as concurrency increases

---

### Interview questions
- Define non-blocking I/O in Node.
- What happens if you call `fs.readFileSync` inside an HTTP request handler?
- I/O-bound vs CPU-bound: what’s the difference and why does it matter?
- How can “async” still become a bottleneck? (thread pool saturation)

---

### Real-world usage (Node.js)
- Prefer async I/O in request paths.
- Stream large payloads (avoid buffering huge bodies).
- Offload CPU-heavy tasks to worker threads or separate services.
- When latency is high, classify the bottleneck:
  - event loop blocked
  - thread pool saturated
  - external dependency slow (DB/upstream)

---

### Practice (do these in Node)
1) Build a tiny HTTP server with:
   - endpoint A uses `readFileSync`
   - endpoint B uses async `readFile`  
   Hit both with concurrency and compare p95 latency.
2) Run many parallel `fs.readFile` calls and observe that throughput eventually flattens (pool saturation).
3) Put a heavy CPU loop in a handler and observe how it blocks even “async” endpoints.

---

### Connections to other topics
- **08 Node internals**: libuv and the thread pool make non-blocking possible.
- **09 Node loop phases**: I/O callbacks are dispatched around poll.
- **12 fs**: filesystem is a key thread-pool-backed async example.
- **19 Performance**: blocking + saturation are major performance failure modes.
