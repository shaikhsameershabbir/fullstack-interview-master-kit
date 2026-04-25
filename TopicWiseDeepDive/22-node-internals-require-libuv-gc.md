## 22) Node Internals (how require works, libuv thread pool, memory & GC)

### Definition (technical)
**Node internals** are the underlying mechanisms of the Node runtime (module resolution/loading/caching, libuv event loop + thread pool, and V8 memory management/GC) that determine observable behavior such as startup cost, I/O concurrency, scheduling order, and memory retention/leaks.

Node internals are where you stop guessing and start explaining.

Teacher truth:
> Internals are not trivia. Internals are debugging power.

This topic focuses on the internals you’ll actually use in real life:
- why `require` behaves like it does (caching + partial initialization)
- why “async” can still become slow (thread pool saturation)
- why memory leaks happen (retained references + GC)

---

### Basic intuition (simple)
- Internals explain why things behave the way they do: startup cost, module caching, I/O concurrency, memory leaks.
- Knowing internals makes production debugging faster because you can form correct hypotheses.

### Internal working (engine level)
#### How `require` works (high level)
- Resolve module path (core module vs file vs `node_modules`)
- Load file contents
- Wrap in a function (CommonJS module wrapper) to provide module scope
- Execute once to produce `module.exports`
- Cache exports for future `require` calls

#### libuv thread pool (again, but deeper)
Some async operations are implemented by:
- queueing work to libuv’s thread pool
- invoking completion callbacks back on the main thread via event loop phases

If the pool is saturated, “async” tasks can become slow under load.

#### Memory management & garbage collection (V8)
- V8 allocates objects on the heap.
- GC reclaims unreachable memory, but GC work can pause JS execution (“stop-the-world” phases).
- Leaks in Node are often **retained references**, not “missing free()”:
  - global caches that only grow
  - event listeners that never removed
  - closures retaining large objects

##### Teacher checkpoint: “GC can’t free what you still reference”
Most Node “leaks” are not mysterious:
- you still have a reference somewhere
- that keeps the object reachable
- reachable objects are not collected

### Edge cases
- Circular dependencies can expose partially-initialized exports (ties to module cache).
- Large object allocations can cause GC pressure and latency spikes.

### Interview questions
- Describe `require` resolution and caching.
- What does the libuv thread pool do?
- What are common causes of memory leaks in Node?

### Real-world usage (Node.js)
- Use heap snapshots and allocation profiling to diagnose leaks.
- Watch for event emitter listener leaks and unbounded caches.
- Tune architecture rather than just tweaking GC flags unless you truly need them.

### Practice (do these in Node)
1) Create two modules with circular dependencies and observe partially initialized exports; explain it.
2) Create an unbounded cache (object keyed by user id) and simulate growth; explain why memory rises.
3) Add many EventEmitter listeners and observe the warning; explain why it happens.
4) Create a closure capturing a large array and keep it globally; explain retention.

### Connections to other topics
- **Module system**: caching + circular deps are internal behaviors you observe externally.
- **Performance**: GC pauses, thread pool saturation, event loop lag.
- **Execution model + closures**: retained lexical environments often explain memory retention.
