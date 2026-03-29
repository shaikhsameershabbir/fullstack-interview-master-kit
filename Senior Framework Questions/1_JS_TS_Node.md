# Module 1: Advanced JavaScript, TypeScript & Node.js Internals

This module covers the profound, "under-the-hood" mechanics of the V8 engine, TypeScript type-level programming, and the Node.js event architecture.

---

## 1️⃣ JavaScript (Core Concepts & Performance)

### 1. What is the JavaScript execution context and the call stack?
**Deep Dive**: 
- **Execution Context**: The environment where JS runs. Contains the *Variable Environment* (memory for variables), *Lexical Environment* (scope chain), and `this` binding. There is a Global Context and functional contexts created on invocation.
- **Call Stack**: The LIFO (Last-In-First-Out) data structure the V8 engine uses to track function execution. When a function is called, its context is pushed to the stack. When it returns, it is popped off. If thousands of recursive calls happen without returning, it causes a "Stack Overflow."

### 2. How does the Event Loop actually work? (Microtasks vs Macrotasks)
**Deep Dive**:
- JS is single-threaded. When async operations occur (fetch, setTimeout), V8 offloads them to Web APIs (or C++ APIs in Node).
- **Macrotask Queue**: Holds callbacks from `setTimeout`, `setInterval`, I/O events.
- **Microtask Queue**: Holds callbacks from `Promises` (.then/.catch) and `MutationObserver`.
- **The Loop**: 
    1. Execute the oldest Macrotask.
    2. Execute *all* pending Microtasks immediately (this can starve the event loop if you infinitely chain Promises).
    3. Render the UI (in the browser).
    4. Repeat.

### 3. What is Debounce vs Throttle at an architectural level?
**Deep Dive**: Both control the rate at which a function executes.
- **Debounce**: "Execute this only after a specified period of silence." If a user is typing in an autocomplete box, debounce waits until they *stop typing* for 300ms before firing the API request. Every keystroke resets the 300ms timer.
- **Throttle**: "Execute this at most once every X milliseconds." If a user is scrolling down a page firing 10,000 events per second, a 100ms throttle ensures the scroll handler executes exactly 10 times a second, providing smooth visual updates without locking the main thread.

### 4. How does Garbage Collection work, and what causes memory leaks?
**Deep Dive**:
- **Mark and Sweep**: V8 periodically starts at the "roots" (the global object and current call stack references). It traces every reachable variable ("Mark"). If an object in memory isn't reachable from the root, it is destroyed ("Sweep").
- **Memory Leaks**: 
    - *Unintended globals*: Forgetting `const` or `let` throws the variable onto the global `window` object, where it can never be GC'd.
    - *Forgotten Timers/Callbacks*: A React component unmounts, but a `setInterval` attached to it wasn't cleared. The closure inside the interval keeps the entire component in memory forever.
    - *Event Listeners*: Adding an `addEventListener` to `document.body` but never calling `removeEventListener`.

---

## 2️⃣ TypeScript (Senior Applications)

### 5. Why use TypeScript in enterprise applications, and what is its fundamental flaw?
**Deep Dive**: 
- **The Value**: TypeScript acts as a massive suite of "free" unit tests for boundary conditions. It prevents 90% of `Cannot read property of undefined` errors at compile time and unlocks profound refactoring safety across a monorepo.
- **The Flaw (Type Erasure)**: TypeScript *does not exist at runtime*. Once compiled, it's just raw JS. A type definition like `interface User { id: number }` offers zero protection if the actual JSON coming from a remote API has `id: "123"` (string). Senior devs bridge this gap using runtime validators (like **Zod** or **Joi**).

### 6. What are Generics and Utility Types?
**Deep Dive**: 
- **Generics**: Functions or classes that accept types as parameterized arguments. `function wrap<T>(value: T): Response<T>`. It allows completely reusable, strictly-typed logic.
- **Utility Types**: Heavy lifting for types.
    - `Partial<T>`: Makes all properties optional (great for `PATCH` requests).
    - `Omit<T, 'id'>`: Creates a new type by stripping out the 'id'.
    - `Record<string, number>`: Equivalent to an object with string keys and number values.

### 7. What are Type Guards and how are they used safely?
**Deep Dive**: Functions that narrow the scope of a union type (`Dog | Cat`).
- *Basic*: `if (typeof x === "string")`
- *Custom (User-Defined Type Guard)*: 
  ```typescript
  function isDog(animal: Dog | Cat): animal is Dog {
      return (animal as Dog).bark !== undefined;
  }
  ```
- *Danger*: The `as` keyword is a lie to the compiler. If `isDog` is implemented incorrectly, the compiler trusts you, leading to runtime crashes.

---

## 3️⃣ Node.js Internals & Architecture

### 8. What is libuv and non-blocking I/O?
**Deep Dive**: 
- V8 handles JS execution, but **libuv** (a C++ library) handles all asynchronous I/O (File system, Network).
- When Node receives an HTTP request hitting the DB, the V8 main thread immediately delegates the network call to libuv and continues processing other users' requests. Once the DB responds, libuv puts the callback into the Event Loop to be executed by V8. This is how a single thread handles 10,000 concurrent connections.

### 9. How do you handle CPU-intensive tasks in Node.js?
**Deep Dive**: 
- *The Problem*: Node.js is single-threaded. If you run a massive cryptographic hash or recursive calculation on the main thread, the entire server freezes. No other user can get an HTTP response.
- *The Solution (Worker Threads)*: Introduced natively in Node. You spawn `new Worker('heavy-task.js')`. This spins up a completely separate V8 isolate on a different CPU core. The main thread passes data to the worker via `postMessage`.

### 10. What is Backpressure in streams?
**Deep Dive**: 
- You are downloading a 5GB file from S3 (Readable Stream) and streaming it directly to the user's browser (Writable Stream).
- If the backend downloads the file from AWS at 1Gbps, but the user's phone writes/downloads it at 5Mbps, the Node.js internal buffer will explode, causing an Out-Of-Memory (OOM) crash.
- **Backpressure**: The writable stream signals `false` (stop). The readable stream pauses automatically. Once the slow client flushes their buffer, the writable stream emits a `drain` event, and the readable stream resumes. Utilizing `.pipe()` handles this automatically.
