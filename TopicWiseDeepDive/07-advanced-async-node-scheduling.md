## 07) Advanced Async (process.nextTick, setImmediate, starvation)

### Definition (technical)
**Advanced scheduling** in Node refers to controlling when callbacks run relative to event loop phases and other queues using primitives like `process.nextTick()` (nextTick queue), `setImmediate()` (check phase), microtasks (Promises), and timers—where misuse can cause **starvation** by preventing lower-priority work (timers/I/O) from being serviced.

Once you understand “microtasks vs macrotasks”, Node gives you a new superpower (and a new way to break production):
- `process.nextTick()`
- `setImmediate()`

Teacher goal:
> You should know **when** to use them, and more importantly **when not to**.

---

### Basic intuition (simple)
- Node has extra scheduling queues beyond the browser mental model.
- `process.nextTick()` is “**before the loop continues**” (very high priority).
- `setImmediate()` is “**soon, but after I/O polling**” (check phase).
- Misuse can cause **starvation**: the process is busy, but important work never gets a turn.

---

### Internal working (engine level)
#### `process.nextTick()` — “run right after the current stack”
Conceptually:
- current stack finishes
- Node drains the **nextTick queue**
- then continues with other microtasks/loop phases

It’s useful for:
- deferring a callback to avoid Zalgo (“sometimes sync, sometimes async”)
- making sure user callbacks run after you finish current internal work

It’s dangerous because:
- you can recursively enqueue `nextTick` and **never let the loop breathe**

##### Predict-output experiment: nextTick priority
Predict output order:

```js
console.log("A");

process.nextTick(() => console.log("nextTick"));
Promise.resolve().then(() => console.log("promise"));
setTimeout(() => console.log("timeout"), 0);

console.log("B");
```

Teacher explanation:
- `A`, `B` are sync
- `nextTick` runs before other continuation queues
- Promises are microtasks
- timeout is later

Exact ordering between `nextTick` and Promise microtasks is Node-specific; the key is:
- **`nextTick` is extremely high priority** and can outrun other work.

#### `setImmediate()` — “run in the check phase”
`setImmediate` callbacks run in the event loop’s **check** phase.

Where it’s useful:
- you’re inside a heavy flow and want to **yield** so I/O callbacks can run
- you want “run soon” behavior that plays well with I/O

#### Starvation patterns (how to freeze your server without an infinite loop)
Starvation means:
- the process is alive
- but important queues never get processed

Classic bug:

```js
function starve() {
  process.nextTick(starve);
}
starve();
```

What happens:
- call stack empties
- nextTick queue keeps refilling
- timers/I/O don’t get a chance

Teacher rule:
> If you schedule “right after now” repeatedly, you can prevent “later” from ever happening.

---

### Edge cases
- Ordering between `setTimeout(fn, 0)` and `setImmediate(fn)` depends on *where you schedule them* (top-level vs inside an I/O callback).
- Don’t build correctness on subtle ordering between nextTick vs Promise microtasks; use them for performance/control, not for logic correctness.

---

### Interview questions
- When would you use `setImmediate` instead of `setTimeout(..., 0)`?
- Why can `process.nextTick` be dangerous?
- What is starvation and what symptoms would you see in a Node server?

---

### Real-world usage (Node.js)
- **Avoid** using `nextTick` for heavy work in servers.
- Use `setImmediate` (or chunking) to keep the event loop responsive during long computations.
- For truly CPU-heavy tasks: use **worker threads** (see `19`), not scheduling tricks.

---

### Practice (do these in Node)
1) Write a script that schedules a timer and then runs a `nextTick` chain. Observe the timer delay and explain it.
2) Implement chunked computation:
   - process 1e7 items, but yield with `setImmediate` every N iterations
   - measure responsiveness (timer still fires)
3) Try scheduling `setTimeout(..., 0)` and `setImmediate(...)` inside an `fs.readFile` callback and observe which runs first.

---

### Connections to other topics
- **05 Event Loop**: these APIs map onto Node’s scheduling model.
- **09 Node loop phases**: `setImmediate` is tied to the check phase.
- **19 Performance**: starvation and event loop blocking are production killers.
