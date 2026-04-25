## 14) HTTP & Servers (Node `http`, request lifecycle, headers/status)

### Definition (technical)
An **HTTP server** is a network service that accepts TCP connections, parses HTTP requests (method, URL, headers, body), executes application logic, and returns HTTP responses (status, headers, body). In Node, this is implemented via the `http` module with request/response streams and event-loop-driven concurrency.

Before Express, before frameworks, before “REST”… there is just this:

> A server is a program that listens for bytes on a port, turns them into an HTTP request, and sends bytes back as a response.

Teacher goal:
> You should be able to mentally trace a request from socket → request object → your handler → response bytes — and understand where Node’s event loop matters.

---

### Basic intuition (simple)
An HTTP server:
- listens on a port
- receives requests (method, path, headers, body)
- sends responses (status, headers, body)

Node can handle many concurrent requests well because:
- most time is spent waiting on I/O (network/DB/disk)
- Node doesn’t block the JS thread while waiting (if you write it correctly)

---

### Internal working (engine level)
#### The request lifecycle (step-by-step)
At a high level, each request goes through:
1) **TCP connection** accepted (or reused via keep-alive)
2) HTTP parser reads bytes and builds:
   - method + URL
   - headers
   - body stream
3) Your JS handler runs on the main thread
4) Response is written (headers + body), often as a stream

Teacher checkpoint:
> The server is “concurrent” because requests are mostly waiting on I/O — not because JS runs in parallel.

#### Status codes (how you communicate outcome)
Think of status codes as “machine-readable meaning”:
- **2xx**: success
- **4xx**: client mistake (bad input, auth missing)
- **5xx**: server fault (bug, dependency down)

Good backend engineers choose status codes intentionally, because:
- retries, caching, and monitoring often depend on them

#### Headers (the control panel of HTTP)
Headers control:
- content type (`Content-Type`)
- caching (`Cache-Control`)
- auth (`Authorization`, cookies)
- compression (`Content-Encoding`)
- connection reuse (`Connection`, keep-alive)

Teacher note:
> Many “random bugs” (CORS, caching, auth) are actually header misunderstandings.

#### Request/response bodies are streams
In Node, request body (`req`) is a readable stream.
Response (`res`) is writable.

That means you can:
- buffer the body (simple, risky for big payloads)
- stream it (scalable)

---

### Edge cases (production mistakes)
#### 1) Forgetting to end the response
If you never call `res.end()` (directly or via helpers), the connection hangs.

#### 2) Blocking the handler blocks everyone
If your handler does heavy sync work:
- event loop is blocked
- all requests slow down

#### 3) Large bodies can kill memory
Always enforce body limits and stream when possible.

---

### Interview questions
- Explain the Node HTTP request lifecycle.
- What’s the difference between buffering a response and streaming it?
- How do status codes influence client behavior?
- Name important headers and what they control.

---

### Real-world usage (Node.js)
- Use Node `http` to learn fundamentals and debug production issues.
- Even if you use Express/Fastify/Nest, the underlying model is the same:
  - streams
  - headers
  - status codes
  - event loop constraints

---

### Practice (do these in Node)
1) Build a server with 2 endpoints:
   - `/sync` does a heavy sync loop
   - `/async` does a `setTimeout` or async I/O  
   Hit both concurrently and observe how `/sync` slows everything.
2) Build `/download` that streams a file with `createReadStream().pipe(res)`.
3) Build `/echo` that streams the request body back to the client.

---

### Connections to other topics
- **05/09 Event loop**: request handlers must keep the call stack short.
- **18 Streams & buffers**: request/response bodies are streams of bytes.
- **15 Express**: frameworks build on this lifecycle using middleware pipelines.
