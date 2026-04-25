## 05) Event Loop Deep Dive (call stack, queues, microtasks vs macrotasks)

### Definition (technical)
The **event loop** is the host/runtime scheduling mechanism that repeatedly selects and executes queued callbacks/tasks when the JavaScript **call stack** is empty, applying priority rules (e.g., **microtasks** before many task queues) and integrating completion notifications from timers, I/O, and other host APIs.

If there is one topic that makes Node.js “click”, it’s the event loop.

You already know JS runs one thing at a time on the main thread.  
So the real question becomes:

> If my code can only do one thing at a time, **how does Node handle thousands of connections** without freezing?

Answer: it runs your sync code on the call stack, and uses the **event loop** to schedule everything else.

---

### Basic intuition (simple)
Picture a restaurant:
- The cook is the JS thread (can cook one dish at a time).
- Orders come in continuously (requests, timers, I/O completions).
- The cook keeps a system to decide what to cook next. That system is the **event loop**.

Core rule:
- Sync code runs **now** on the call stack.
- Async callbacks run **later**, but only when the call stack is empty.

---

### Internal working (engine level)
#### The minimum model: 3 pieces you must not mix up
1) **Call Stack**: what is executing right now.
2) **Host/runtime**: provides timers, I/O, networking (browser APIs in browsers, libuv in Node).
3) **Queues**: waiting areas for callbacks.

Teacher checkpoint:
> The event loop does *not* “run your code in parallel”. It decides **when** callbacks are allowed to enter the call stack.

#### Macrotask vs Microtask (the priority system)
The event loop uses priority.

**Microtasks** (higher priority):
- Promise reactions (`then`, `catch`, `finally`)
- `queueMicrotask`

**Macrotasks** (lower priority, host-dependent):
- timers (`setTimeout`, `setInterval`)
- many I/O callbacks (varies by environment)

High-level rule that explains most output-order puzzles:
1) Run synchronous code until stack empty
2) Drain microtasks (often completely)
3) Run next macrotask callback
4) Repeat

#### Predict-output experiment #1: Promise vs setTimeout
Predict the output order:

```js
console.log("A");

setTimeout(() => console.log("timeout"), 0);

Promise.resolve().then(() => console.log("promise"));

console.log("B");
```

Teacher explanation:
- `A`, `B` are sync → run first
- Promise `.then` is a microtask → runs before timers
- timer callback is macrotask → runs after microtasks

Expected order:
1) A
2) B
3) promise
4) timeout

##### Checkpoint
Say it in one sentence:
- Why does `"promise"` print before `"timeout"`?

#### Predict-output experiment #2: “async/await doesn’t block”
Predict order:

```js
async function run() {
  console.log("1");
  await Promise.resolve();
  console.log("2");
}

console.log("0");
run();
console.log("3");
```

Expected order:
0, 1, 3, 2

Teacher explanation:
- `await` pauses the async function and schedules the continuation as a microtask.

---

### Edge cases (real traps)
#### 1) Microtask starvation
If you keep scheduling microtasks, you can delay timers and I/O.

Example pattern:
- a microtask schedules another microtask
- repeat forever

Symptoms in Node servers:
- timers fire late
- latency spikes
- process feels “alive but stuck”

#### 2) `setTimeout(fn, 0)` is not “run immediately”
It means “run later when the loop gets to timers and the stack is empty.”
Under load, “0ms” can become “50ms+”.

#### 3) Unhandled promise rejections
Unhandled rejections can:
- crash the process (depending on Node settings/version)
- or cause silent inconsistency

Teacher rule: treat rejections like thrown errors—handle them.

---

### Interview questions
- Why does Promise `.then` run before `setTimeout(..., 0)`?
- What is the difference between microtasks and macrotasks?
- What does it mean to “block the event loop”?
- Explain microtask starvation and why it’s dangerous.

---

### Real-world usage (Node.js)
#### 1) “Non-blocking server” is just “event loop friendly”
Node handles many requests because:
- it doesn’t block on I/O (I/O is handled by the runtime)
- it only blocks when *your JS* hogs the call stack

#### 2) Debugging latency
When latency is high, ask:
- is the event loop blocked by sync CPU work?
- are we drowning in microtasks?
- is the thread pool saturated? (next topics)

#### 3) Designing APIs
Your goal is to keep the call stack short:
- async I/O
- streaming for large payloads
- move CPU-heavy work off main thread (worker threads)

---

### Practice (do these in Node)
1) Write 3 logs: one sync, one Promise, one timer. Predict order, then run.
2) Create a microtask chain (`Promise.resolve().then(() => Promise.resolve().then(...))`) and observe how it delays a timer.
3) Simulate event loop blocking with a big loop inside a server handler and observe how other requests slow down.

---

### Connections to other topics
- **01 Execution Model**: callbacks only enter the stack when it’s empty.
- **06 Async Patterns**: Promises/async-await schedule microtasks.
- **07 Advanced Async**: Node has `process.nextTick` and `setImmediate` which affect scheduling.
- **19 Performance**: “event loop blocking” is the #1 practical Node performance issue.
