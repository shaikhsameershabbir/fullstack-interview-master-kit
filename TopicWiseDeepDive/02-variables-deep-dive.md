## 02) Variables Deep Dive (var / let / const, hoisting, TDZ, memory)

### Definition (technical)

A **variable** in JavaScript is a **binding** (identifier → value slot) stored in an environment record of a lexical environment. Declaration forms (`var`/`let`/`const`) differ in **scope**, **creation/initialization timing** (hoisting + TDZ), and **reassignment rules**, which determines observable runtime behavior.

Let’s start with the most common lie you’ll hear in JavaScript:

> “Hoisting means JavaScript moves code to the top.”

Nothing is moved. If you learn the real story, you automatically understand:

- why `var` feels “weird”
- why `let`/`const` throw TDZ errors
- why some bugs only appear at runtime
- why closures can keep memory alive in Node

---

### Basic intuition (simple)

When JS runs a scope (global or function), it does two things:

1. **Registers** names (bindings) during the **creation phase**
2. **Executes** your code line-by-line during the **execution phase**

So the best way to learn variables is to stop thinking:

- “a variable is created when I write the line”

And start thinking:

- “a variable binding exists earlier, and becomes usable at a specific moment”

That moment depends on whether you used `var`, `let`, or `const`.

---

### Internal working (engine level)

#### Hoisting (the correct definition)

Hoisting is the *observable behavior* caused by:

- **binding registration** during creation phase

The engine creates a mapping from **identifier name → storage location** before it starts executing line-by-line.

Teacher lens: for each declaration type, ask these 3 questions:

1. **Where** is it scoped? (function / block / module)
2. **When** is it created? (creation phase vs later)
3. **When** is it initialized (made usable)?

#### `var` (the “legacy behavior”)

**Scope**

- `var` is **function-scoped** (not block-scoped).

**Creation phase**

- A `var` binding is created **and initialized immediately to `undefined`**.

**Execution phase**

- Later, when your assignment line runs, the value changes from `undefined` → actual value.

That’s why you can see:

```js
console.log(a); // undefined (not ReferenceError)
var a = 10;
```

What’s happening is not “code moved.” It’s:

- binding exists early
- initialized to `undefined`
- so reading it is allowed (but risky)

**Why `var` causes bugs**

- redeclaration is allowed
- function scope + “initialized early” produces confusing behavior in large codebases

#### `let` (block scope + safety)

**Scope**

- `let` is **block-scoped**.

**Creation phase**

- binding is created (so the name exists in the scope)
- but it is **not initialized**

That “exists but not initialized” period is the TDZ.

```js
console.log(x); // ReferenceError
let x = 1;
```

Teacher point: that error is a feature. It catches wrong ordering early.

#### TDZ (Temporal Dead Zone)

The TDZ is the time window where the binding:

- is already registered in the scope
- but is still **uninitialized**

So any access (read or write) throws a `ReferenceError`.

Why does TDZ exist?

- It makes code safer by preventing you from using a variable before the declaration line “makes it real.”
- It gives engines more predictable semantics for block-scoped bindings.

Important nuance (your handwritten note nailed this):

- People say “`let` is not hoisted”
- Better: `**let` is hoisted (binding exists) but not initialized** (huge difference)

#### `const` (like `let`, but stricter)

`const` follows the same TDZ behavior as `let`, with one extra rule:

- it must be **initialized at the declaration**

```js
const a = 10; // OK
// const b;   // SyntaxError (must initialize)
```

And remember:

- `const` protects the **binding** (the variable name), not the **value’s mutability**.

```js
const user = { name: "a" };
user.name = "b"; // OK (object mutated)
// user = {};    // TypeError (rebinding)
```

#### Visual timeline (the teacher-friendly mental model)

Imagine time flowing top → bottom in your file.

`**var**`

- creation phase: binding created + initialized to `undefined`
- execution phase: assignment happens later

`**let` / `const**`

- creation phase: binding created
- TDZ: binding exists but unusable
- at declaration line: initialization happens

##### Checkpoint

Explain in one sentence why this prints `undefined`:

```js
console.log(a);
var a = 1;
```

And why this throws:

```js
console.log(b);
let b = 1;
```

#### Memory allocation (stack vs heap) — with the exact nuance people miss

You’ll hear:

- “primitives go on stack”
- “objects go on heap”

That’s a decent start, but the real story that helps you debug Node memory issues is:

- **Stack**: call frames + many fixed-size values + **references (pointers)**
- **Heap**: objects/arrays/functions and other dynamically-sized structures

So in:

```js
let x = { big: "data" };
```

You can think:

- `x` is a binding in the current lexical environment.
- the “value” of `x` is a **reference** stored in frame-ish/stack-ish data.
- the object `{ big: "data" }` itself lives in the **heap**.

This matters because:

- closures can keep references alive
- references keep heap objects reachable
- reachable heap objects don’t get garbage collected

---

### Edge cases (the ones that confuse everyone)

#### 1) `typeof` special-cases undeclared identifiers

This surprises people:

```js
typeof totallyNotDeclared; // "undefined"
```

But:

- accessing `totallyNotDeclared` directly throws `ReferenceError`.

#### 2) `var` ignores block scope

```js
if (true) {
  var a = 1;
}
console.log(a); // 1
```

With `let`, it behaves like normal block scoping.

#### 3) Shadowing changes which binding you get

```js
let x = 1;
{
  let x = 2;
  console.log(x); // 2
}
console.log(x);   // 1
```

Nearest binding wins.

---

### Interview questions (with the “good answer shape”)

- Explain hoisting without saying “moves code”.
- Why does `let` throw `ReferenceError` but `var` often gives `undefined`?
- What is TDZ and why does it exist?
- Explain `const` with objects (binding vs mutability).
- Where do objects live (stack vs heap) and why does that matter for leaks?

---

### Real-world usage (Node.js)

#### 1) Prefer `let`/`const` in servers

`var` redeclaration + function scope is a bug magnet in long-lived services.

#### 2) TDZ errors help you catch wrong ordering early

Especially in large modules where import order and initialization order matters.

#### 3) Memory leaks are often “variable behavior” + “closure behavior”

If you keep request objects in module-level arrays/caches, or capture big objects in closures:

- they stay reachable
- heap grows
- GC works harder
- latency worsens

---

### Practice (do these in Node)

1. Predict the output, then run:

```js
console.log(a);
var a = 10;
console.log(a);
```

1. Predict the error type, then run:

```js
console.log(b);
let b = 10;
```

1. Write a loop that pushes functions into an array using `var` (closure trap). Then fix it with `let`. Explain why the fix works using “block scope + new binding each iteration”.

---

### Connections to other topics

- **01 Execution Model**: hoisting + TDZ are consequences of the **creation phase**.
- **03 Scope & Lexical Environment**: `let/const` are block-scoped because they belong to the block’s lexical environment record.
- **Closures**: captured bindings can keep heap objects alive (real Node memory leak pattern).
- **04 `this`**: `this` is a binding too, but it’s resolved by call-site rules, not hoisting rules.

