## 11) Module System (CommonJS vs ES Modules, caching, circular deps)

### Definition (technical)
A **module system** defines how code is packaged into isolated units with explicit imports/exports, how those units are resolved/loaded/executed, and how results are cached. In Node, **CommonJS** uses runtime `require()` with `module.exports`, while **ESM** uses static `import`/`export` with different linking/execution semantics; both can exhibit caching and circular-dependency partial initialization behaviors.

You can’t build real Node apps in one file.  
So the next “deep dive” question is:

> When I `require()` or `import`, what *actually happens*? And why do circular deps and “singleton modules” appear?

This topic teaches you modules the way a teacher would: as a runtime mechanism, not just syntax.

---

### Basic intuition (simple)
A module system answers:
- How do I split code into files?
- How do I reuse code?
- How does the runtime load, execute, and cache files?

Node supports two module systems:
- **CommonJS (CJS)**: `require()` + `module.exports`
- **ES Modules (ESM)**: `import` + `export`

The most important real-world behavior:
> Modules are typically executed once, then **cached**.  
> That means imports can share state.

---

### Internal working (engine level)
#### CommonJS mental model (CJS)
When you do `require("./x")`, Node roughly does:
1) resolve the path (core module vs file vs `node_modules`)
2) load the file contents
3) wrap it in a function (module scope wrapper)
4) execute it once to produce `module.exports`
5) cache it for future `require` calls

Teacher checkpoint:
> “Require is synchronous” means your JS thread waits while Node loads and executes the module.

#### ESM mental model (ES Modules)
ESM is designed for static analysis:
- imports are resolved in a structured way
- bindings are “live” (imported values are views into exports, not copies)
- loading/execution is staged differently than CJS

You don’t need all the details on day 1. What you *do* need:
- ESM has different initialization semantics
- mixing CJS and ESM has interop rules that can surprise you

#### Module caching (the “singleton” effect)
Once a module is loaded:
- Node caches its exports
- future imports/require return the cached object

Teacher-style consequence:
- module-level variables behave like “singletons per process”

Predict what happens:

```js
// counter.js
let n = 0;
module.exports.inc = () => ++n;
```

If 3 different files `require("./counter")` and call `inc()`, do they share `n`?
- Yes, because the module is cached and executed once.

#### Circular dependencies (why “undefined exports” happen)
Circular dependency means:
- A loads B
- B loads A

But modules run in steps:
- A starts executing, then tries to load B
- B starts executing, then tries to load A
- Node gives B the *partially initialized* exports of A (because A hasn’t finished executing)

So B may see `undefined` for exports that A sets later.

Teacher rule:
> Circular deps aren’t “impossible”. They’re “dangerous because of partial initialization.”

---

### Edge cases
- Mixing CJS and ESM leads to confusing default export behavior.
- Module caching can become hidden global state if you store mutable data at module scope.
- Circular deps can hide until runtime and then fail in weird ways.

---

### Interview questions
- Compare CommonJS vs ESM in Node (syntax + loading + caching).
- What does module caching imply for state and side effects?
- Explain circular dependencies and partial initialization.

---

### Real-world usage (Node.js)
- Prefer ESM for modern codebases unless the ecosystem forces CJS.
- Avoid module-level mutable state in servers unless it’s intentionally a process-wide singleton (and bounded).
- Break circular deps with:
  - dependency inversion (move shared types/helpers)
  - lazy requires (last resort)
  - refactoring modules into layers

---

### Practice (do these in Node)
1) Build a tiny `counter` module and require it from two files; prove caching by showing shared state.
2) Create a circular dependency between two modules and observe `undefined` exports; explain “partial initialization.”
3) Convert a small CJS module to ESM and observe import/export differences.

---

### Connections to other topics
- **01 Execution Model**: modules have initialization order; execution happens during loading.
- **22 Node internals**: deeper “how require works” and memory/GC implications.
- **19 Performance**: startup time and module side effects affect cold start.
