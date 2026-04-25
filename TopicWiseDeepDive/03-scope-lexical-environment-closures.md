## 03) Scope, Lexical Environment & Closures

### Definition (technical)
**Scope** is the set of bindings accessible at a program point. A **lexical environment** is the runtime structure holding an environment record plus an outer reference, forming the **scope chain** used for identifier resolution. A **closure** is a function that retains access to bindings from its defining lexical environment, even after the outer execution context has returned.

If you’ve ever said:
- “Why can’t I access this variable here?”
- “Why does my callback print the wrong value?”
- “How is `count` still alive when the function returned?”

…you were really asking about **scope**, **lexical environments**, and **closures**.

This topic is the bridge between “I can write JS” and “I understand JS like an engine.”

---

### Basic intuition (simple)
Let’s define the three words in plain English:

- **Scope**: the region of code where a name is visible.
- **Lexical**: “based on where the code is written.”
- **Closure**: a function carrying a backpack of references to variables from its surrounding scope.

One sentence that explains 90% of JS behavior:

> JavaScript resolves variables by walking **upward** through lexical environments created by where the code is written.

---

### Internal working (engine level)
#### 1) Lexical environment: what it contains
Every execution context has a Lexical Environment with two parts:
- **Environment Record**: the actual bindings (variables, parameters, function declarations)
- **Outer reference**: a pointer to the parent lexical environment

The outer reference is not a metaphor. Conceptually it’s a real link the engine follows.

#### 2) Scope chain: how lookup happens (step-by-step)
When the engine needs the value of `x`, it does this:
1) Look for `x` in the **current** environment record
2) If not found, follow the **outer reference**
3) Repeat until global
4) If still not found, throw `ReferenceError`

Teacher rule:
- JS lookup is **nearest-first**
- and lookup moves **upward only**

##### Mini experiment: nearest binding wins (shadowing)
Predict output:

```js
let x = "global";

function demo() {
  let x = "function";
  if (true) {
    let x = "block";
    console.log("inside block:", x);
  }
  console.log("inside function:", x);
}

demo();
console.log("global:", x);
```

You should get:
- block
- function
- global

If you can explain *why* in terms of “nearest environment record”, you’ve got it.

#### 3) Closures: the “backpack” model (what your note said perfectly)
Let’s teach closure the way a teacher would:

When a function is created, it remembers:
- the code inside it
- and the lexical environment where it was created (outer reference)

Now your classic counter:

```js
function counter() {
  let count = 0;

  return function () {
    count++;
    console.log(count);
  };
}

const inc = counter();
inc(); // 1
inc(); // 2
inc(); // 3
```

**Step-by-step explanation**
1) Calling `counter()` creates an execution context with a lexical environment containing `count`.
2) `counter()` returns the inner function.
3) The inner function keeps an outer reference to the `counter()` lexical environment.
4) Even after `counter()` returns (call frame popped), the lexical environment can stay alive **because something still references it** (the returned function).

So closure is not “magic memory.” It’s normal reachability:
- if something can still reach `count`, it stays alive

##### Checkpoint
Answer in one sentence:
- Why isn’t `count` destroyed when `counter()` returns?

Correct shape:
- “Because the returned function still references the lexical environment where `count` lives.”

---

### Edge cases (where closures bite)
#### 1) The loop + closure trap (classic)
Predict output:

```js
const fns = [];
for (var i = 0; i < 3; i++) {
  fns.push(() => i);
}
console.log(fns[0](), fns[1](), fns[2]());
```

Why it prints `3 3 3`:
- `var` is function-scoped, so there is **one binding** of `i`
- all closures point to the same binding
- by the time you call them, `i` is 3

Fix with `let`:

```js
const fns = [];
for (let i = 0; i < 3; i++) {
  fns.push(() => i);
}
console.log(fns[0](), fns[1](), fns[2]());
```

Why it works:
- each iteration gets a **fresh block-scoped binding** of `i`
- closures capture different bindings

#### 2) Accidental retention (Node memory leaks)
This is the “adult closure problem”:
- closures can keep large objects alive unintentionally

Example pattern:
- you attach an event listener or timer callback that closes over a big request object
- listener is never removed
- request object stays reachable forever

Teacher rule:
> A memory leak in JS is often: “I kept a reference somewhere.”  
> Closures are one of the easiest ways to keep references without noticing.

---

### Interview questions
- Define lexical scope and scope chain.
- What is a closure, and why does it happen?
- Explain the `var` loop closure bug and two fixes.
- How can closures cause memory leaks in Node?

---

### Real-world usage (Node.js)
- **Request handlers** rely on closures constantly (capturing validated user id, correlation ids, config).
- **Async code**: callbacks and promise handlers close over state; bugs often come from capturing the wrong binding.
- **Leak hunting**: long-lived closures in timers, event listeners, module-level caches are common culprits.

---

### Practice (do these in Node)
1) Write `makeAdder(x)` that returns a function `(y) => x + y`. Explain why `x` is still available.
2) Recreate the loop-closure bug with `var`, then fix it with `let`. Explain using “one binding vs many bindings.”
3) Build a small EventEmitter example where a listener closes over an object. Remove the listener and explain how that helps GC.

---

### Connections to other topics
- **01 Execution Model**: lexical environment is part of an execution context.
- **02 Variables / TDZ**: block-scoped bindings (`let/const`) change closure behavior (especially loops).
- **05 Event Loop**: async callbacks are almost always closures capturing state.
- **04 `this`**: closures preserve variables, but `this` is a call-site binding (don’t treat it like captured state).
