## 01) Execution Model (How JS actually runs)

### Definition (technical)

The **execution model** of JavaScript is the runtime mechanism by which the engine creates **execution contexts**, manages them on the **call stack**, resolves identifiers via **lexical environments/scope chains**, and advances program evaluation according to the language and host scheduling rules.

You’ll hear people say “JavaScript is single-threaded.” That sentence is *useful*, but it’s incomplete.  
The real question (especially for Node.js) is:

> If JS can only run one thing at a time, **how does it keep track of what’s running now, what runs next, and where variables live?**

This file teaches you the “engine-thinking” answer.

---

### Basic intuition (simple)

Think of JS execution like a classroom:

- The teacher (the engine) creates a **workspace** before teaching a lesson. That workspace is the **Execution Context**.
- The teacher can only talk to **one student at a time** (single main thread).
- So the teacher keeps a **stack of papers** on the desk saying “what I’m doing right now.” That stack is the **Call Stack** (LIFO).

If you internalize just one line:

- **Execution Context = workspace**
- **Call Stack = execution order**

---

### Internal working (engine level)

#### 1) Execution Context (EC) = workspace

Before code “runs”, the engine needs a container to manage:

- **bindings** for variables (names → storage)
- **function declarations**
- **scope references** (how identifiers resolve)
- the current `**this`** value
- **arguments/parameters** during calls

Types of execution context you’ll hear about:

- **Global Execution Context (GEC)**: created when the script is loaded.
- **Function Execution Context (FEC)**: created each time a function is *called*; removed when it returns.
- **Eval Execution Context**: created by `eval()` (avoid in real code).

Teacher tip: you can ignore eval EC for now. If you understand GEC + FEC deeply, you understand 95% of what matters.

#### 2) Every EC has two phases (this is where “hoisting” comes from)

Most confusion in JS happens because people imagine the engine “starts running line 1 immediately.”  
Engines don’t do that.

**Phase A — Creation phase (setup / registration)**

- Engine scans the scope.
- It **registers** variables and functions in memory structures (bindings exist now).
- It decides the initial `**this`** value for that context.

**Phase B — Execution phase (run time)**

- Now it runs the code **top to bottom**.
- Variable assignments happen.
- Function calls create new contexts.
- Returns destroy the top context and resume the previous one.

✅ This is the correct mental model of hoisting:

> Hoisting is not “moving code.”  
> Hoisting is the *observable behavior* created by **creation phase registration**.

#### 3) The Call Stack (control flow)

The Call Stack is a LIFO stack of “what we’re currently inside”.

Let’s do a guided walk-through. First, predict:

```js
function one() {
  console.log("one");
}

function two() {
  one();
  console.log("two");
}

two();
```

**Your prediction**: What prints first?

**Teacher explanation (stack story)**

- Script starts → push **GEC**
- We call `two()` → push **FEC(two)**
- Inside `two()`, we call `one()` → push **FEC(one)**
- `one()` finishes → pop **FEC(one)**
- We continue inside `two()` → print `"two"` → return → pop **FEC(two)**
- Program ends → pop **GEC**

Output:

- `one`
- `two`

##### Checkpoint (answer without looking)

- Why does `one()` run completely before `"two"` prints?
- What data structure enforces that rule?

#### 4) Lexical Environment (what “exists” inside a context)

Each execution context has a **Lexical Environment**. Think of it like a notebook with:

- **Environment Record**: “these are my local bindings”
- **Outer Reference**: “if I don’t have the name, ask my parent scope”

This outer reference is how scopes are linked.

#### 5) Scope Chain (how variables are found)

When the engine sees an identifier like `count`, it searches:

1. current environment record  
2. outer reference → parent environment  
3. …repeat until global

Two crucial rules:

- It searches **upward only** (child scopes are not searched).
- It stops at the **first match** (nearest binding wins).

##### Mini experiment: “upward only”

Predict what happens:

```js
function parent() {
  function child() {
    console.log(x);
  }
  child();
}

parent();
let x = 10;
```

If you said “it prints 10” — that’s the trap.  
This will throw (because `x` is in TDZ until initialized). We’ll study TDZ in topic `02`, but notice how execution order + scope rules *combine*.

---

### Edge cases (where people crash)

#### 1) Stack overflow (deep recursion)

Too many nested calls:

- `RangeError: Maximum call stack size exceeded`

Teacher rule: recursion is fine when depth is controlled. Unbounded recursion is a bug.

#### 2) `eval()` breaks your mental model

It can introduce names dynamically and disables many optimizations. Avoid it.

#### 3) “Global `this`” differs by environment

- Browser scripts often bind `this` to `window`.
- Node has module scoping and strict-mode differences.
We’ll cover `this` deeply in `04` and the module wrapper in `11`/`22`.

---

### Interview questions (with “teacher expectations”)

- Explain **execution context** and its two phases.
- Explain **hoisting** without saying “JS moves code”.
- What’s the difference between **call stack** and **scope chain**?
- Why do **closures** work after the parent function returns? (Answer: lexical environment + outer reference)

---

### Real-world usage (Node.js)

#### 1) Reading stack traces

Every error stack trace is basically the call stack printed for humans.

#### 2) Debugging performance

If you do heavy synchronous work in a request handler:

- the call stack stays busy
- the event loop can’t process other requests
- latency spikes

This is why Node performance topics always start with the execution model.

#### 3) Memory leaks via retained scope

If a function returns an inner function that closes over big objects, those objects can stay alive longer than you expect.

---

### Practice (do these in Node)

1. Write a function `a()` that calls `b()` that calls `c()`. Throw an error in `c()` and observe the stack trace order. Explain it using the call stack.
2. Create a recursion bug intentionally (e.g., function calls itself without base case). Observe the stack overflow error and explain why it happens.
3. Create a closure that captures a big array and keep the returned function in a global variable. Explain why memory might not be freed.

---

### Connections to other topics (the “map”)

- **02 Variables / Hoisting / TDZ**: hoisting behavior is a direct result of the **creation phase**.
- **03 Scope & Lexical Environment**: lexical env + outer reference is the foundation of the **scope chain** and **closures**.
- **04 `this*`*: `this` is part of execution context state, but the value is decided by invocation rules.
- **05 Event Loop**: event loop decisions only happen when the call stack becomes empty.

