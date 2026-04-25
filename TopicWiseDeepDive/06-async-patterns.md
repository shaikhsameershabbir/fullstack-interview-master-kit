## 06) Async Patterns (callbacks → promises → async/await)

### Definition (technical)
**Asynchronous programming** in JS/Node structures computations whose results become available later via callbacks or promise continuations scheduled by the host/event loop. **Callbacks** register functions for later invocation, **Promises** model eventual completion as a state machine, and **async/await** expresses promise-based control flow via suspension and microtask-based resumption.

Async is not “doing two things at once” in JavaScript.  
Async is: **don’t block the main thread while waiting**.

This topic is about *how humans structure async code*, and what the runtime is really doing underneath.

We’ll learn it in evolution order:
1) callbacks (works, but messy)
2) promises (composable)
3) async/await (promise syntax in “sync style”)

---

### Basic intuition (simple)
When you call an async API, you’re basically saying:
- “Start this work”
- “Come back later with the result”

The patterns differ in how they represent “come back later”.

---

### Internal working (engine level)
#### 1) Callbacks: “call me when you’re done”
Classic Node-style callback (error-first):

```js
fs.readFile("a.txt", "utf8", (err, data) => {
  if (err) return console.error(err);
  console.log(data);
});
```

Why callbacks hurt at scale:
- nested control flow becomes unreadable (“callback pyramid”)
- error propagation is manual and inconsistent
- composing sequential vs parallel work is awkward

##### Checkpoint
Why is error handling harder with callbacks than with synchronous code?

#### 2) Promises: “a value that will exist later”
A Promise is a state machine:
- **pending** → **fulfilled(value)** or **rejected(reason)**

What matters for the event loop:
- `.then` handlers run as **microtasks** (after current sync code finishes)

Promise chain example:

```js
doA()
  .then((a) => doB(a))
  .then((b) => doC(b))
  .catch((err) => handle(err));
```

Teacher note: chaining works because returning a promise “links” the chain.

#### 3) async/await: “promise code that reads like sync”
An `async function` always returns a Promise.

```js
async function main() {
  const a = await doA();
  const b = await doB(a);
  return doC(b);
}
```

What `await` does (engine thinking):
- it *pauses* the async function
- yields control back to the event loop
- schedules the continuation as a microtask when the awaited promise settles

##### Big misunderstanding
`await` does **not** block the thread.  
It blocks *only that async function’s continuation*.

---

### Edge cases (important in Node)
#### 1) `try/catch` and missing `await`
This catches:

```js
try {
  await Promise.reject(new Error("boom"));
} catch (e) {
  console.log("caught");
}
```

This often does **not** catch what you think:

```js
try {
  Promise.reject(new Error("boom")); // missing await/return
} catch (e) {
  console.log("not caught");
}
```

Teacher rule:
- if you want the error to be catchable here, you must `await` (or `return`) the promise.

#### 2) “Sequential vs parallel” awaits (performance)
Sequential:

```js
await a();
await b();
```

Parallel (start both first):

```js
const p1 = a();
const p2 = b();
await p1;
await p2;
```

Even better:

```js
await Promise.all([a(), b()]);
```

But only use parallel when tasks are independent.

#### 3) Mixing callbacks + promises (double-completion bugs)
If you wrap a callback API into a Promise incorrectly, you can:
- resolve twice
- reject after resolve
- swallow errors

---

### Interview questions
- Compare callbacks vs Promises vs async/await.
- What does it mean that Promise handlers are “microtasks”?
- Does `await` block the main thread? Explain precisely.
- `Promise.all` vs `Promise.allSettled`: when and why?

---

### Real-world usage (Node.js)
- Prefer Promise APIs: `fs/promises`, modern DB clients, fetch-like HTTP clients.
- Use `Promise.all` for independent I/O to reduce latency.
- Always handle rejections in servers:
  - return errors consistently
  - avoid unhandled rejections

Teacher mindset in production:
> Your job isn’t “use async/await.”  
> Your job is “keep the event loop responsive and keep failures handled.”

---

### Practice (do these in Node)
1) Write the same flow using:
   - callbacks
   - promise chaining
   - async/await
   Compare readability and error handling.
2) Write a function that starts 3 independent requests and uses `Promise.all`.
3) Create a bug by forgetting `await` inside `try/catch`, then fix it and explain why.

---

### Connections to other topics
- **05 Event Loop**: Promises/async-await schedule microtasks; ordering matters.
- **07 Advanced Async**: `nextTick` and `setImmediate` can change scheduling behavior in Node.
- **10 Non-blocking I/O**: async patterns are the userland interface to I/O.
