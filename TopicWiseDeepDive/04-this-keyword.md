## 04) `this` Keyword (binding rules, arrow vs normal, call/apply/bind)

### Definition (technical)
`this` is a **runtime binding** provided to function execution that refers to the receiver/context for that call. For normal functions it is primarily determined by the **call-site** (default/implicit/explicit/`new` binding rules), while arrow functions **lexically capture** `this` from the surrounding scope and do not create their own `this` binding.

If scope/closures answer “**where do variables come from?**”, then `this` answers:

> “When this function runs, **which object am I talking about right now?**”

The reason `this` feels confusing is simple:
- variables use **lexical rules** (where code is written)
- `this` mostly uses **call-site rules** (how code is called)

Let’s learn it like a teacher: we’ll use a small set of rules, then we’ll test them with “predict output” exercises.

---

### Basic intuition (simple)
- `this` is a **runtime binding**.
- In most cases, it is decided by the **call-site** (the line that calls the function).
- Arrow functions are special: they don’t create their own `this`; they **inherit** (`lexically capture`) `this` from the surrounding scope.

Teacher sentence:
> Don’t ask “where is the function written?” Ask “**how is it called?**”

---

### Internal working (engine level)
Think of `this` as a value stored in the current execution context when a function is invoked.

There are 4 main binding rules. Learn them in this priority order:

#### Rule 1: `new` binding (constructor call)
If a function is called with `new`, `this` is a **newly created object**.

```js
function User(name) {
  this.name = name;
}

const u = new User("A");
console.log(u.name); // "A"
```

Teacher note: `new` is usually the strongest binding rule.

#### Rule 2: Explicit binding (`call`, `apply`, `bind`)
If you explicitly set `this`, the engine respects it.

```js
function sayName() {
  return this.name;
}

const a = { name: "A" };

console.log(sayName.call(a)); // "A"
console.log(sayName.apply(a)); // "A"
const bound = sayName.bind(a);
console.log(bound()); // "A"
```

Quick memory aid:
- `call(obj, a, b)` → args listed
- `apply(obj, [a, b])` → args array
- `bind(obj)` → returns a new function with fixed `this`

#### Rule 3: Implicit binding (method call)
If you call it as `obj.fn()`, then `this = obj`.

```js
const user = {
  name: "A",
  say() {
    return this.name;
  },
};

console.log(user.say()); // "A"
```

##### The “lost `this`” trap
Predict what happens:

```js
const say = user.say;
console.log(say());
```

Teacher explanation:
- `say()` is now a **plain function call**, not `obj.fn()`.
- so implicit binding is gone
- `this` becomes default-bound (next rule)

#### Rule 4: Default binding (plain function call)
If you call `fn()` without `obj.` and without explicit binding:
- in **strict mode**, `this` is `undefined`
- in **sloppy mode**, `this` may become a global object (environment-dependent)

This is why good Node code often uses strict mode (or ESM/modules which are effectively strict) and avoids relying on default binding.

#### Arrow functions (lexical `this`)
Arrow functions don’t have their own `this`. They use the `this` from the surrounding scope.

This is why they’re popular inside methods:

```js
const obj = {
  x: 1,
  later() {
    setTimeout(() => {
      console.log(this.x); // 1
    }, 0);
  },
};

obj.later();
```

Teacher warning:
- don’t use arrows when you *need* dynamic `this` (like methods meant to be reused with different receivers)

---

### Edge cases (the ones that bite in real apps)
#### 1) Callbacks often lose `this`

```js
const obj = {
  x: 1,
  getX() { return this.x; }
};

setTimeout(obj.getX, 0); // `this` lost
```

Fixes:
- `bind`: `setTimeout(obj.getX.bind(obj), 0)`
- wrapper: `setTimeout(() => obj.getX(), 0)`

#### 2) `this` is not “closure state”
Your note said it well:
- closures preserve variables via lexical environment
- `this` is normally rebound per call

So don’t try to “store state in this” unless you’re intentionally using objects/classes.

---

### Interview questions
- Explain the 4 binding rules for `this` (and which wins).
- Why does `const fn = obj.method; fn()` behave differently from `obj.method()`?
- Arrow vs normal function `this`: what changes and why?
- When would you use `bind` in production code?

---

### Real-world usage (Node.js)
- **EventEmitter listeners** sometimes use `this` to refer to the emitter (depends on API). Don’t guess—test or read docs.
- In services/classes, prefer patterns that avoid “mystery `this`”:
  - pass dependencies explicitly
  - avoid extracting methods unless bound
- Arrow methods on classes trade convenience for memory (each instance gets its own function). Prototype methods are shared but require correct binding in callbacks.

---

### Practice (do these in Node)
1) Predict output, then run:

```js
const obj = {
  name: "X",
  say() { return this.name; },
};
console.log(obj.say());
const f = obj.say;
console.log(f());
```

Explain both lines using binding rules.

2) Fix the “lost this” case using `bind` and using a wrapper arrow.
3) Write a small constructor function with `new`, then call it without `new` and observe the difference (in strict mode).

---

### Connections to other topics
- **01 Execution Model**: `this` lives in execution context state, but its value is decided by invocation.
- **03 Closures**: arrow functions feel like closures for `this` because they capture it lexically.
- **11 Module system**: top-level `this` differs between CommonJS/ESM and strict mode.
