## 🧠 Node.js + JavaScript Deep Dive (Sequenced Notes)

This repo is divided into levels (Beginner → Expert).  
Each topic has its own **numbered** note file so you can follow a clean learning sequence.

### 🟢 LEVEL 1 — JavaScript Foundations (Engine Thinking)
Goal: Understand how JS actually runs.

#### 1.1 Execution Model
- [x] Notes: [`topics/01-execution-model.md`](topics/01-execution-model.md)
- [x] Execution Context (Global + Function)
- [x] Call Stack
- [x] Creation vs Execution Phase

#### 1.2 Variables Deep Dive
- [x] Notes: [`topics/02-variables-deep-dive.md`](topics/02-variables-deep-dive.md)
- [x] var, let, const
- [x] Hoisting (real meaning)
- [x] Temporal Dead Zone (TDZ)
- [x] Memory allocation (stack vs heap)

#### 1.3 Scope & Lexical Environment
- [x] Notes: [`topics/03-scope-lexical-environment-closures.md`](topics/03-scope-lexical-environment-closures.md)
- [ ] Scope chain
- [ ] Lexical environment internals
- [ ] Closures (VERY IMPORTANT)

#### 1.4 this Keyword
- [x] Notes: [`topics/04-this-keyword.md`](topics/04-this-keyword.md)
- [ ] Binding rules
- [ ] Arrow vs normal functions
- [ ] call, apply, bind

### 🟡 LEVEL 2 — Asynchronous JavaScript (CORE OF NODE)
Goal: Understand non-blocking behavior.

#### 2.1 Event Loop Deep Dive
- [x] Notes: [`topics/05-event-loop-deep-dive.md`](topics/05-event-loop-deep-dive.md)
- [ ] Call stack vs callback queue
- [ ] Microtask vs Macrotask
- [ ] Promise queue

#### 2.2 Async Patterns
- [x] Notes: [`topics/06-async-patterns.md`](topics/06-async-patterns.md)
- [ ] Callbacks → Problems
- [ ] Promises → Internals
- [ ] Async/Await → Sugar vs reality

#### 2.3 Advanced Async
- [x] Notes: [`topics/07-advanced-async-node-scheduling.md`](topics/07-advanced-async-node-scheduling.md)
- [ ] process.nextTick()
- [ ] setImmediate()
- [ ] Starvation problems

### 🔵 LEVEL 3 — Node.js Runtime Internals
Goal: Understand what makes Node different.

#### 3.1 What is Node.js Internally
- [x] Notes: [`topics/08-what-is-node-internally.md`](topics/08-what-is-node-internally.md)
- [ ] V8 Engine
- [ ] Libuv
- [ ] Thread pool

#### 3.2 Event Loop (Node vs Browser)
- [x] Notes: [`topics/09-node-vs-browser-event-loop.md`](topics/09-node-vs-browser-event-loop.md)
- [ ] All phases in Node
- [ ] Poll phase deep dive

#### 3.3 Non-blocking I/O
- [x] Notes: [`topics/10-non-blocking-io.md`](topics/10-non-blocking-io.md)
- [ ] How file/network operations work
- [ ] Why Node is fast

### 🟣 LEVEL 4 — Core Node Modules (System Level)
Goal: Work with OS & low-level features.

#### 4.1 Module System
- [x] Notes: [`topics/11-module-system.md`](topics/11-module-system.md)
- [ ] CommonJS vs ES Modules
- [ ] Module caching
- [ ] Circular dependencies

#### 4.2 File System (fs)
- [x] Notes: [`topics/12-file-system-fs.md`](topics/12-file-system-fs.md)
- [ ] Sync vs Async
- [ ] Streams vs readFile

#### 4.3 Core Modules
- [x] Notes: [`topics/13-core-modules-path-os-events-buffer-crypto.md`](topics/13-core-modules-path-os-events-buffer-crypto.md)
- [ ] path, os, events
- [ ] buffer (VERY IMPORTANT)
- [ ] crypto

### 🔶 LEVEL 5 — Backend Development (Real World)
Goal: Build production APIs.

#### 5.1 HTTP & Servers
- [x] Notes: [`topics/14-http-and-servers.md`](topics/14-http-and-servers.md)
- [ ] Creating server using http
- [ ] Request lifecycle
- [ ] Headers, status codes

#### 5.2 Framework (Express.js)
- [x] Notes: [`topics/15-express-routing-middleware-errors.md`](topics/15-express-routing-middleware-errors.md)
- [ ] Routing
- [ ] Middleware (deep dive)
- [ ] Error handling

#### 5.3 Databases
- [x] Notes: [`topics/16-databases-mongo-sql-transactions-indexing.md`](topics/16-databases-mongo-sql-transactions-indexing.md)
- [ ] MongoDB + Mongoose
- [ ] SQL basics
- [ ] Transactions & indexing

#### 5.4 Authentication
- [x] Notes: [`topics/17-authentication-jwt-vs-sessions-security.md`](topics/17-authentication-jwt-vs-sessions-security.md)
- [ ] JWT vs Sessions
- [ ] Security basics

### 🔴 LEVEL 6 — Advanced Node (Senior Level)
Goal: Think like a backend architect.

#### 6.1 Streams & Buffers
- [x] Notes: [`topics/18-streams-and-buffers.md`](topics/18-streams-and-buffers.md)
- [ ] Stream lifecycle
- [ ] Backpressure
- [ ] Pipe mechanism

#### 6.2 Performance Optimization
- [x] Notes: [`topics/19-performance-optimization.md`](topics/19-performance-optimization.md)
- [ ] Event loop blocking
- [ ] Clustering
- [ ] Worker Threads

#### 6.3 Scaling Systems
- [x] Notes: [`topics/20-scaling-systems.md`](topics/20-scaling-systems.md)
- [ ] Load balancing
- [ ] Stateless apps
- [ ] Caching (Redis)

#### 6.4 Message Queues
- [x] Notes: [`topics/21-message-queues.md`](topics/21-message-queues.md)
- [ ] Kafka / RabbitMQ
- [ ] Pub/Sub architecture

### ⚫ LEVEL 7 — Expert (Internals + Architecture)
Goal: Stand out as senior engineer.

#### 7.1 Node Internals
- [x] Notes: [`topics/22-node-internals-require-libuv-gc.md`](topics/22-node-internals-require-libuv-gc.md)
- [ ] How require works
- [ ] Libuv thread pool deep dive
- [ ] Memory management & GC

#### 7.2 System Design
- [x] Notes: [`topics/23-system-design.md`](topics/23-system-design.md)
- [ ] API design at scale
- [ ] Rate limiting
- [ ] DB scaling

#### 7.3 Architecture Patterns
- [x] Notes: [`topics/24-architecture-patterns.md`](topics/24-architecture-patterns.md)
- [ ] MVC / Clean Architecture
- [ ] Microservices vs Monolith
- [ ] Event-driven systems

### 🧩 How we study each topic (consistent structure)
For EVERY topic file:
- Basic intuition (simple)
- Internal working (engine level)
- Edge cases
- Interview questions
- Real-world usage (Node.js)
