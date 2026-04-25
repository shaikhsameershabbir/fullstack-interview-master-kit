## 09) Event Loop: Node vs Browser (phases, poll phase)

### Definition (technical)
The **Node event loop** is libuv’s phased scheduling loop (timers → poll → check → …) that dispatches callbacks from different sources, whereas the **browser event loop** is the browser host’s task scheduling model; both integrate microtasks, but they differ in phases, I/O sources, and available scheduling APIs.

If you learned the event loop from browser tutorials, Node will surprise you.

In browsers, people talk about:
- “task queue”
- “microtask queue”

In Node, there’s an extra layer:
- libuv phases (timers, poll, check, …)
- a special `process.nextTick` queue

Teacher goal:
> You should be able to explain *why* `setImmediate` exists, what “poll phase” means, and why ordering can differ depending on where you schedule things.

---

### Basic intuition (simple)
- Browser event loop = the browser’s scheduling model.
- Node event loop = **libuv’s** scheduling model.
- Both run JS on a single call stack, but they differ in:
  - phases
  - APIs
  - I/O sources

Think: same idea (schedule work), different engine (host runtime).

---

### Internal working (engine level)
#### Node event loop phases (conceptual)
Node’s loop is commonly described with these phases:
- **timers**: run `setTimeout` / `setInterval` callbacks whose time has elapsed
- **pending callbacks**: some system-level callbacks
- **idle/prepare**: internal
- **poll**: retrieve I/O events; execute I/O callbacks; can wait for I/O
- **check**: run `setImmediate` callbacks
- **close callbacks**: e.g., socket close events

And “side queues” that can run between steps:
- **microtasks** (Promises)
- **`process.nextTick`** queue (very high priority)

Teacher checkpoint:
> Node isn’t “Promise queue + timeout queue”. It’s phases + queues.

#### The poll phase (why it matters)
Poll is where Node deals with I/O:
- if I/O callbacks are ready, run them
- if nothing is ready, Node may **wait** (block in poll) for new I/O

But it won’t wait forever if something else is scheduled. Roughly:
- if timers are due, go run timers
- if `setImmediate` is queued, go to check phase
- otherwise, poll can wait for I/O (efficient)

This is why `setImmediate` is useful:
- it gives you a way to run “soon” **after I/O poll**, not just “after N ms”.

---

### Edge cases (ordering puzzles)
#### `setTimeout(..., 0)` vs `setImmediate(...)`
Teacher truth:
- At top-level, ordering can be unpredictable across runs.
- Inside an I/O callback, `setImmediate` often wins because you’re already in/near poll → check.

Try this inside an I/O callback to feel it:

```js
const fs = require("node:fs");

fs.readFile(__filename, () => {
  setTimeout(() => console.log("timeout"), 0);
  setImmediate(() => console.log("immediate"));
});
```

#### `process.nextTick()` starvation
As you learned in `07`, nextTick can outrun the phases and delay I/O.

---

### Interview questions
- List Node’s event loop phases (high level) and what runs where.
- Where does `setImmediate` run and why does it exist?
- What is the poll phase responsible for?
- Explain starvation in Node with `nextTick` or microtasks.

---

### Real-world usage (Node.js)
- If timers are late in production, it’s often:
  - event loop blocked
  - microtask/nextTick starvation
  - too much I/O callback work (poll congestion)
- Understanding phases helps you reason about “why this callback didn’t run yet.”

---

### Practice (do these in Node)
1) Run a top-level script with both `setTimeout(..., 0)` and `setImmediate(...)` and observe ordering.
2) Run the same scheduling inside an `fs.readFile` callback and observe ordering changes.
3) Create a nextTick chain and see how it delays a timer; explain which “queue” is starving which “phase”.

---

### Connections to other topics
- **07 Advanced Async**: `setImmediate` ties to check phase; `nextTick` is special priority.
- **10 Non-blocking I/O**: I/O completion callbacks are dispatched around poll.
- **19 Performance**: slow timers and latency spikes often come from loop congestion.
