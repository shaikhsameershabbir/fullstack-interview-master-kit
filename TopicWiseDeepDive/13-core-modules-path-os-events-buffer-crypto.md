## 13) Core Modules (path, os, events, buffer, crypto)

### Definition (technical)
Node **core modules** are built-in, versioned APIs shipped with Node that expose OS-level functionality (paths, system info), event-driven primitives (EventEmitter), binary data handling (Buffer), and cryptographic primitives (hashing/signing/encryption), forming the runtime’s standard library for backend development.

Node core modules are like your “standard library.”  
But this topic isn’t about memorizing APIs—it’s about understanding the *ideas* you’ll use everywhere in backend work:
- events
- bytes (Buffer)
- crypto boundaries (what to do / what not to do)
- safe path handling

Teacher goal:
> After this, Node’s built-in modules won’t feel like random utilities—they’ll feel like building blocks.

---

### Basic intuition (simple)
- **`path`**: build safe file paths (don’t hand-concatenate strings).
- **`os`**: observe machine capabilities and limits.
- **`events`**: event-driven programming (publish/subscribe).
- **`Buffer`**: raw bytes (files, sockets, crypto).
- **`crypto`**: hashing/signing/encryption primitives (use carefully).

---

### Internal working (engine level)
#### `events` (EventEmitter) — the “callback hub”
Many Node APIs are event-based (streams, sockets, servers).

Mental model:
- `.on(event, fn)` registers listeners
- `.emit(event, ...args)` calls listeners later (synchronously at emit time)

Mini example:

```js
const { EventEmitter } = require("node:events");

const bus = new EventEmitter();
bus.on("data", (x) => console.log("got", x));
bus.emit("data", 123);
```

Teacher checkpoint:
> `emit` is usually synchronous. That means a slow listener can slow the emitter.

Common failure:
- adding listeners and never removing them → memory growth

#### `Buffer` — bytes, not text
Strings are *text*. Buffers are *bytes*.

Why you need buffers:
- file reads return bytes
- network packets are bytes
- crypto works on bytes

Mini experiment:

```js
const b = Buffer.from("hi", "utf8");
console.log(b);          // <Buffer ...>
console.log(b.length);   // 2
console.log(b.toString("utf8")); // "hi"
```

Teacher checkpoint:
> Encoding is how bytes become text (and back). If you choose wrong encoding, data can corrupt.

#### `crypto` — powerful and dangerous
What you typically do in backend apps:
- password hashing (bcrypt/argon2/scrypt via libs; not raw encryption)
- signing tokens (JWT)
- random bytes (secure IDs)

What you should *not* do:
- invent your own encryption protocol
- reuse nonces/IVs incorrectly
- store secrets in plain text

Teacher rule:
> Use proven libraries and patterns for high-level crypto. Use Node `crypto` primitives only when you know exactly what you’re doing.

#### `path` — “don’t break Windows / don’t allow path traversal”
Use `path.join`/`path.resolve` instead of manual concatenation:
- handles separators correctly (`/` vs `\`)
- prevents weird edge cases like double slashes

#### `os` — observe environment (useful for ops)
Use `os.cpus()`, `os.totalmem()`, etc., for:
- sizing worker pools
- debugging container limits
- logging environment info

---

### Edge cases
- Buffer encoding mistakes (`utf8`, `base64`, `hex`) can corrupt data.
- EventEmitter leaks: forgetting to remove listeners keeps objects alive.
- Crypto misuse: rolling your own encryption is dangerous.

---

### Interview questions
- What is a Buffer and why not use string for everything?
- How does EventEmitter work and what can go wrong?
- Why is `path.join` preferred over string concatenation?
- When should you avoid using raw crypto primitives directly?

---

### Real-world usage (Node.js)
- Streams + buffers power file uploads/downloads efficiently.
- EventEmitter patterns show up in HTTP servers, sockets, internal architecture.
- Crypto shows up in auth, password hashing, signing, secure IDs.

---

### Practice (do these in Node)
1) Build an EventEmitter that emits “tick” events; add/remove listeners and observe memory/behavior.
2) Read a file as Buffer, convert to string, then base64 encode; explain what each step means.
3) Generate secure random bytes for an ID and represent it as hex.
4) Use `path.join` to safely build a path; show how naive string concatenation can break cross-platform.

---

### Connections to other topics
- **18 Streams & Buffers**: buffers are the unit of streaming I/O.
- **19 Performance**: buffers + crypto affect memory/CPU; event emitters can leak.
- **17 Authentication**: crypto primitives appear in hashing/signing workflows.
