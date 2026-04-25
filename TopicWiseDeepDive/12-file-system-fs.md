## 12) File System (`fs`) (sync vs async, streams vs readFile)

### Definition (technical)
The Node `fs` module provides APIs for interacting with the filesystem. **Synchronous** APIs block the event loop until the OS call completes, while **asynchronous** APIs schedule work and deliver results later via callbacks/promises (often using libuv’s thread pool). **Streams** provide incremental I/O with backpressure for large or continuous data.

Filesystem is one of the best places to *feel* what “non-blocking” means in Node.

Teacher framing:
> Your server isn’t slow because disk is slow.  
> Your server is slow because you made the **event loop wait** for disk.

This topic teaches you when to use:
- sync APIs (rarely)
- async APIs (usually)
- streams (whenever data is big or continuous)

---

### Basic intuition (simple)
- Disk I/O is slow compared to CPU operations.
- If you block the JS thread waiting for disk, everything else waits too.
- Async fs APIs let Node keep responding while I/O happens elsewhere.

For large files:
- `readFile` loads everything into memory → can blow up RAM
- streams process chunks → stable memory usage

---

### Internal working (engine level)
#### Sync vs async: what changes?
**Sync**
- JS calls `readFileSync`
- JS thread blocks until the OS returns data
- no other requests/callbacks can be processed in that time

**Async**
- JS calls `readFile`
- Node schedules work (often via libuv thread pool for fs)
- JS thread continues
- later, callback runs on the event loop

##### Predict-output experiment
Predict output:

```js
const fs = require("node:fs");

console.log("A");
fs.readFile(__filename, "utf8", () => console.log("file done"));
console.log("B");
```

Expected:
- A
- B
- file done

Because the read completes later.

#### `readFile` vs streams (the memory story)
**`readFile`**
- loads full file into memory (Buffer/string)
- great for small files
- dangerous for large files under concurrency

**Streams**
- read in chunks
- can start processing before the whole file is loaded
- support **backpressure**

Teacher checkpoint:
> Streams aren’t “faster” by default. They’re “safer and more scalable” because they control memory and flow.

---

### Edge cases (production traps)
#### 1) Thread pool saturation
Many fs async operations can queue up behind the thread pool.
Symptom:
- not blocked event loop, but requests still slow under high concurrency.

#### 2) Encoding mistakes
If you forget `utf8`, you’ll get a Buffer.
That’s not “wrong”—it just means you’re dealing with bytes.

#### 3) File descriptor/resource leaks
With streams, always handle errors; otherwise resources can stay open longer than you expect.

---

### Interview questions
- Why is `readFileSync` dangerous in an HTTP server?
- When do you prefer streams over `readFile`?
- What is backpressure and why does it matter?

---

### Real-world usage (Node.js)
- Prefer `fs/promises` + async/await for readability.
- Use streams for:
  - uploads/downloads
  - piping data (e.g., read → transform → write)
  - log processing and ETL
- Enforce size limits; don’t accept unbounded file reads in request paths.

---

### Practice (do these in Node)
1) Build a small server endpoint that returns a file:
   - version A uses `readFile` and `res.end(data)`
   - version B uses `createReadStream().pipe(res)`
   Compare memory and latency with a large file.
2) Run many concurrent `readFile` calls and observe performance flattening (pool).
3) Write a stream copy script and handle `error` events; explain why it matters.

---

### Connections to other topics
- **10 Non-blocking I/O**: filesystem is a key example of thread-pool-backed async work.
- **18 Streams & Buffers**: deeper stream lifecycle + backpressure later.
- **19 Performance**: thread pool saturation and memory spikes affect latency.
