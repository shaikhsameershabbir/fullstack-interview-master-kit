# JavaScript Advanced – Mid-Level Developer Interview Reference

> Deep explanations, internals, and real-world context for each concept.

---

## 🔗 Section 1: JavaScript Advanced Concepts

---

### 1. What is the Prototype Chain?

**Definition:** The prototype chain is JavaScript's fundamental mechanism for **inheritance and property lookup**. Every object has an internal `[[Prototype]]` slot that points to another object (its prototype). When you access a property on an object, JavaScript searches the object itself first, then traverses up the chain via `[[Prototype]]` links until it finds it or reaches `null`.

**Internal representation:**
```
myObject → myObject.__proto__ → Object.prototype → null
```

**How property lookup works:**
```js
const animal = {
  breathe() { return "breathing"; }
};

const dog = Object.create(animal); // dog.__proto__ = animal
dog.bark = function() { return "woof!"; };

const rex = Object.create(dog); // rex.__proto__ = dog
rex.name = "Rex";

// Property lookup chain for rex.breathe():
// 1. Look on rex              → not found
// 2. Look on rex.__proto__ (dog)        → not found
// 3. Look on dog.__proto__ (animal)     → FOUND! ✅
rex.breathe(); // "breathing"

// Full chain:
// rex → dog → animal → Object.prototype → null

// Visualising the chain
console.log(Object.getPrototypeOf(rex) === dog);    // true
console.log(Object.getPrototypeOf(dog) === animal); // true
console.log(Object.getPrototypeOf(animal) === Object.prototype); // true
console.log(Object.getPrototypeOf(Object.prototype)); // null (end of chain)
```

**Constructor function prototype chain:**
```js
function Person(name) {
  this.name = name;
}
Person.prototype.greet = function() {
  return `Hello, I'm ${this.name}`;
};

const alice = new Person("Alice");

// When 'new' is used:
// 1. Creates a new object {}
// 2. Sets __proto__ = Person.prototype
// 3. Runs constructor with 'this' = new object
// 4. Returns the new object

// Chain for alice:
// alice → Person.prototype → Object.prototype → null
alice.greet();             // found on Person.prototype
alice.hasOwnProperty("name"); // found on Object.prototype

// Own property check
alice.hasOwnProperty("name");  // true  (own property)
alice.hasOwnProperty("greet"); // false (inherited)
"greet" in alice;              // true  (searches chain)
```

**Key methods:**
```js
Object.getPrototypeOf(obj)        // get [[Prototype]] (safe)
Object.setPrototypeOf(obj, proto) // set [[Prototype]] (avoid — slow)
Object.create(proto)              // create object with given prototype
obj.hasOwnProperty(key)           // check OWN property (not inherited)
key in obj                        // checks own + chain
```

---

### 2. How Does JavaScript Inheritance Work?

**Definition:** JavaScript uses **prototypal inheritance** — objects inherit directly from other objects via the prototype chain. Unlike classical OOP (Java, C++), there are no "true" classes; `class` syntax is syntactic sugar over the existing prototype mechanism.

**Three levels of understanding:**

#### Level 1 — `Object.create()` (purest form):
```js
const vehicleProto = {
  describe() { return `${this.brand} ${this.model}`; },
  start()    { return `${this.brand} engine started`; }
};

function createCar(brand, model, doors) {
  const car = Object.create(vehicleProto); // set prototype
  car.brand = brand;
  car.model = model;
  car.doors = doors;
  return car;
}

const tesla = createCar("Tesla", "Model 3", 4);
tesla.describe(); // "Tesla Model 3" — found via prototype
```

#### Level 2 — Constructor Functions (pre-ES6):
```js
function Animal(name, sound) {
  this.name = name;
  this.sound = sound;
}
Animal.prototype.speak = function() {
  return `${this.name} says ${this.sound}`;
};

function Dog(name, breed) {
  Animal.call(this, name, "woof"); // inherit Animal's properties
  this.breed = breed;
}
// Set up prototype chain: Dog.prototype → Animal.prototype
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog; // fix broken constructor reference

Dog.prototype.fetch = function() {
  return `${this.name} fetches the ball!`;
};

const rex = new Dog("Rex", "Labrador");
rex.speak();   // "Rex says woof"   — inherited from Animal
rex.fetch();   // "Rex fetches!"    — own method
rex instanceof Dog;    // true
rex instanceof Animal; // true
```

#### Level 3 — `class` syntax (ES6, recommended):
```js
class Animal {
  #sound; // private field (ES2022)

  constructor(name, sound) {
    this.name = name;
    this.#sound = sound;
  }

  speak() {
    return `${this.name} says ${this.#sound}`;
  }

  // Static method — called on class, not instances
  static kingdom() { return "Animalia"; }

  // Getter
  get info() { return `${this.name} (${this.#sound})`; }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name, "woof"); // MUST call super before using 'this'
    this.breed = breed;
  }

  // Override parent method
  speak() {
    return `${super.speak()} and wags tail`; // call parent method
  }

  fetch() { return `${this.name} fetches!`; }
}

class ServiceDog extends Dog {
  constructor(name, breed, duty) {
    super(name, breed);
    this.duty = duty;
  }

  describe() {
    return `${this.name} is a service dog: ${this.duty}`;
  }
}

const buddy = new ServiceDog("Buddy", "Golden Retriever", "Guide");
buddy.speak();    // "Buddy says woof and wags tail"
buddy.describe(); // "Buddy is a service dog: Guide"
buddy.fetch();    // "Buddy fetches!"
Animal.kingdom(); // "Animalia"
```

**Important:**
- `class` doesn't create true classes — it's still prototypal under the hood
- `typeof Dog === "function"` proves it
- `extends` sets up the prototype chain automatically
- `super()` in constructor calls the parent constructor
- Private fields (`#field`) are truly inaccessible from outside

---

### 3. What is Event Delegation?

**Definition:** Event delegation is a pattern where you attach **a single event listener to a parent element** to manage events for **all its children** — including ones added dynamically in the future. It works because of **event bubbling**: events propagate from the target element up through its ancestors.

**How event bubbling works:**
```
User clicks <button> inside <li> inside <ul>
Event fires on <button>
  → bubbles up to <li>
    → bubbles up to <ul>   ← your single listener is here
      → bubbles up to <body>
        → bubbles up to <html>
          → reaches <document>
```

**Without event delegation (naive approach):**
```js
// ❌ Problems: many listeners, memory waste, doesn't work for dynamic elements
const buttons = document.querySelectorAll(".action-btn");
buttons.forEach(btn => {
  btn.addEventListener("click", handleClick); // one listener per element
});

// Adding new buttons dynamically?
const newBtn = document.createElement("button");
newBtn.className = "action-btn";
list.appendChild(newBtn);
// newBtn has NO listener! You'd have to add it manually.
```

**With event delegation:**
```js
// ✅ One listener handles all children — current AND future
const list = document.getElementById("task-list");

list.addEventListener("click", function(event) {
  // event.target = the ACTUAL element clicked
  // event.currentTarget = the element the listener is on (list)

  const target = event.target;

  // Handle different child elements
  if (target.matches(".delete-btn")) {
    const li = target.closest("li");
    li.remove();
  }

  if (target.matches(".complete-btn")) {
    const li = target.closest("li");
    li.classList.toggle("completed");
  }

  if (target.matches(".edit-btn")) {
    const taskId = target.dataset.id;
    openEditor(taskId);
  }
});

// Dynamically added items automatically work!
function addTask(text) {
  const li = document.createElement("li");
  li.innerHTML = `
    ${text}
    <button class="delete-btn">Delete</button>
    <button class="complete-btn">Done</button>
    <button class="edit-btn" data-id="${Date.now()}">Edit</button>
  `;
  list.appendChild(li); // automatically works with the existing listener!
}
```

**Advanced — generic event delegation utility:**
```js
function delegate(parentEl, eventType, selector, handler) {
  parentEl.addEventListener(eventType, function(event) {
    const target = event.target.closest(selector);
    if (target && parentEl.contains(target)) {
      handler.call(target, event);
    }
  });
}

delegate(document.body, "click", ".delete-btn", function(e) {
  this.closest(".card").remove();
});
```

**Benefits:** Memory efficiency, handles dynamic elements, single source of truth for handling.
**Gotchas:** `event.stopPropagation()` on a child will prevent the delegate from seeing it. Not suitable for events that don't bubble (e.g., `focus`, `blur` — use `focusin`/`focusout` instead).

---

### 4. What is Debounce vs Throttle?

**Both are techniques to control the rate at which a function executes in response to frequent events.**

#### Debounce
**Definition:** Debounce ensures a function is only called **after a specified delay has passed since the last invocation**. If called again before delay ends, the timer resets. Like an elevator door that waits N seconds after the last person enters before closing.

```js
/**
 * Debounce — delays execution until after 'delay' ms of inactivity
 * @param {Function} fn - function to debounce
 * @param {number} delay - wait time in ms
 * @returns {Function} - debounced version
 */
function debounce(fn, delay) {
  let timerId = null;

  return function(...args) {
    // Clear previous timer — reset the countdown
    clearTimeout(timerId);

    // Set a new timer
    timerId = setTimeout(() => {
      fn.apply(this, args);
      timerId = null;
    }, delay);
  };
}

// Use case: Search input — don't fire API for every keystroke
const searchInput = document.getElementById("search");

const fetchResults = debounce(async (query) => {
  console.log("Fetching:", query); // only fires 300ms after last keystroke
  const data = await fetch(`/api/search?q=${query}`).then(r => r.json());
  renderResults(data);
}, 300);

searchInput.addEventListener("input", e => fetchResults(e.target.value));

// User types: "j" → "ja" → "jav" → "java" (quickly)
// Only ONE request fires: for "java" (after 300ms of silence)
```

#### Throttle
**Definition:** Throttle ensures a function fires **at most once per specified interval**, regardless of how many times it is triggered. Like a water tap that only drips every N ms, no matter how much water pressure there is.

```js
/**
 * Throttle — ensures function fires at most once per 'interval' ms
 * @param {Function} fn - function to throttle
 * @param {number} interval - minimum ms between calls
 * @returns {Function} - throttled version
 */
function throttle(fn, interval) {
  let lastCallTime = 0;
  let timerId = null;

  return function(...args) {
    const now = Date.now();
    const remaining = interval - (now - lastCallTime);

    if (remaining <= 0) {
      // Enough time has passed — fire immediately
      clearTimeout(timerId);
      lastCallTime = now;
      fn.apply(this, args);
    } else {
      // Schedule for when interval completes
      clearTimeout(timerId);
      timerId = setTimeout(() => {
        lastCallTime = Date.now();
        fn.apply(this, args);
      }, remaining);
    }
  };
}

// Use case: scroll tracking — update position at most every 100ms
const handleScroll = throttle(() => {
  console.log("Scroll Y:", window.scrollY); // fires max every 100ms
  updateProgressBar(window.scrollY);
}, 100);

window.addEventListener("scroll", handleScroll);

// Use case: button click prevention (no double-submit)
const submitBtn = document.getElementById("submit");
submitBtn.addEventListener("click", throttle(submitForm, 2000)); // max once every 2s
```

#### Comparison:
| Feature | Debounce | Throttle |
|---|---|---|
| Fires when | After inactivity period | At regular intervals |
| If event fires 10x quickly | Fires once (at end) | Fires ~N times (at intervals) |
| Use case | Search, resize listener, form validation | Scroll, mousemove, window resize, API polling |
| Guarantees execution | Only after silence | At consistent intervals |

```
Event: ████████████████████████████
        ^events firing rapidly

Debounce (300ms):             ▼ (fires once after silence)
Throttle (300ms):   ▼   ▼   ▼   ▼ (fires at intervals)
```

---

### 5. How Does Garbage Collection Work in JavaScript?

**Definition:** Garbage collection (GC) is the **automatic memory management** process by which the JavaScript engine identifies and frees memory occupied by objects that are no longer reachable (referenced) by the running program.

**JavaScript uses two main algorithms:**

#### 1. Reference Counting (older, simpler — not used by modern engines):
```js
// Idea: count how many references point to each object
// When count reaches 0 → free the memory

let obj1 = { name: "Alice" }; // ref count = 1
let obj2 = obj1;              // ref count = 2
obj1 = null;                  // ref count = 1
obj2 = null;                  // ref count = 0 → garbage collected

// Problem: circular references!
let a = {};
let b = {};
a.ref = b; // a references b
b.ref = a; // b references a — circular!
a = null;
b = null;
// Both have ref count = 1 (each other) → NEVER collected!
// This is why reference counting isn't used alone.
```

#### 2. Mark-and-Sweep (modern engines — V8, SpiderMonkey):
```js
// Idea: "reachability" — if you can reach an object from roots, it's alive

// Roots:
// - Currently executing function's local variables
// - Global variables
// - The call stack

// Algorithm:
// Phase 1 MARK: Start from roots, traverse ALL references recursively
//               Mark every reachable object as "alive"
// Phase 2 SWEEP: Scan all memory; anything NOT marked → free it

// Example:
function example() {
  let user = { name: "Alice", order: { id: 1 } }; // marked (reachable via user)
  let admin = { role: "admin" };                    // marked (reachable via admin)
  admin = null;            // admin now unreachable → will be swept
  return user.name;
  // user.order still reachable via user (which is returned's scope)
}
```

**V8's Generational GC:**
```
New Space (Young Generation) — small, collected frequently (minor GC)
  New objects live here first
  Most objects die young (functions' local vars)
  Survivors promoted to Old Space after 2 GCs

Old Space (Old Generation) — larger, collected less often (major GC)
  Long-lived objects (DOM nodes, caches, module-level vars)
  Major GC is "stop-the-world" — pauses execution (V8 minimizes this)

V8 optimizations:
- Incremental marking — spread GC across multiple frames (avoid jank)
- Concurrent marking — mark in background thread while JS runs
- Orinoco (parallel) GC — use multiple CPU cores for GC
```

**What triggers GC:**
- Heap approaching max limit
- After a certain number of allocations (heuristic)
- You can hint (not control) via `global.gc()` in Node.js with `--expose-gc` flag

---

### 6. What Causes Memory Leaks in JavaScript?

**Definition:** A memory leak occurs when your program allocates memory and then **fails to release it** even though it's no longer needed, causing memory usage to grow over time and eventually degrade performance or crash the app.

#### Cause 1 — Forgotten Global Variables:
```js
function leakyFunction() {
  // ❌ No 'let/const/var' — creates global variable!
  leakedData = new Array(1000000); // lives forever on 'window'
}

// Fix: always use 'const' or 'let'
function fixedFunction() {
  const data = new Array(1000000); // garbage collected when function returns
}

// Accidental global via 'this' in non-strict mode
function leak() {
  this.huge = new Array(1000000); // 'this' is window in non-strict mode!
}
leak(); // window.huge = [...] — memory leak

// Fix: use "use strict"; or class methods
```

#### Cause 2 — Forgotten setInterval / setTimeout:
```js
// ❌ Interval keeps the callback (and everything it closes over) alive
let heavyData = fetchLargeDataset();

const intervalId = setInterval(() => {
  process(heavyData); // closes over heavyData — keeps it in memory!
}, 1000);

// If you navigate away without clearing the interval:
// heavyData is NEVER garbage collected!

// Fix: always clear intervals/timeouts when done
function setupPolling() {
  const data = fetchData();
  const id = setInterval(() => process(data), 1000);

  // Return cleanup function
  return () => {
    clearInterval(id);
    // data can now be GC'd
  };
}
const cleanup = setupPolling();
// When component unmounts / page unloads:
cleanup();
```

#### Cause 3 — Detached DOM Nodes:
```js
// ❌ DOM node removed from document but still referenced in JS
let detachedNode;
function createNode() {
  const div = document.createElement("div");
  div.id = "myDiv";
  document.body.appendChild(div);
  detachedNode = div; // reference stored outside
}

document.body.removeChild(document.getElementById("myDiv"));
// Node is removed from DOM but 'detachedNode' still holds it in memory!

// Fix: nullify references when DOM elements are removed
function removeNode() {
  const node = document.getElementById("myDiv");
  node.parentNode?.removeChild(node);
  detachedNode = null; // allow GC
}
```

#### Cause 4 — Closures Holding Large Data:
```js
// ❌ Inner function keeps outer function's scope alive
function processData() {
  const hugeArray = new Array(1_000_000).fill("data"); // 1M elements!

  return function() {
    // This inner function closes over hugeArray
    // Even if only the inner function is needed, hugeArray stays!
    return "done";
  };
}
const fn = processData(); // hugeArray is stuck in memory as long as fn exists
```

#### Cause 5 — Event Listeners Not Removed:
```js
// ❌ In a Single Page App, listeners accumulate across route changes
class Component {
  mount() {
    this.handleClick = () => this.onClick();
    document.body.addEventListener("click", this.handleClick);
    window.addEventListener("resize", this.onResize.bind(this));
  }

  // If unmount doesn't clean up:
  unmount() {
    // ❌ Missing cleanup → listeners + component instance stay in memory!
  }
}

// ✅ Fix: always remove listeners in cleanup
unmount() {
  document.body.removeEventListener("click", this.handleClick);
  window.removeEventListener("resize", this.onResize);
}
```

#### Cause 6 — Caches Without Eviction:
```js
// ❌ Unbounded cache grows forever
const cache = new Map();
function getUser(id) {
  if (cache.has(id)) return cache.get(id);
  const user = fetchUser(id); // expensive
  cache.set(id, user); // cache grows forever!
  return user;
}

// ✅ Fix: use LRU cache with max size or WeakMap (auto-evicts)
const cache = new Map();
const MAX_CACHE = 100;
function getUser(id) {
  if (cache.has(id)) {
    const val = cache.get(id);
    cache.delete(id);
    cache.set(id, val); // move to end (LRU)
    return val;
  }
  if (cache.size >= MAX_CACHE) {
    cache.delete(cache.keys().next().value); // evict oldest
  }
  const user = fetchUser(id);
  cache.set(id, user);
  return user;
}
```

**Debugging memory leaks:**
- Chrome DevTools → Memory tab → Heap Snapshot
- Compare snapshots before and after suspected leak triggers
- Look for growing "Detached DOM tree" or array sizes

---

### 7. What are WeakMap and WeakSet?

**Definition:** `WeakMap` and `WeakSet` are collection types where the references to their **keys (WeakMap) or values (WeakSet) are "weak"** — they don't prevent JavaScript's garbage collector from reclaiming objects when there are no other strong references to them.

#### WeakMap:
```js
// WeakMap<Object, any>
// - Keys MUST be objects (not primitives)
// - Keys are weakly referenced (GC can collect them)
// - NOT iterable, no .size, no .forEach, no .keys()
// Use: associating private data or metadata with objects without preventing GC

const weakMap = new WeakMap();

let user = { id: 1, name: "Alice" };
weakMap.set(user, { sessionToken: "abc123", loginTime: Date.now() });

weakMap.get(user);  // { sessionToken: "abc123", ... }
weakMap.has(user);  // true
weakMap.delete(user);

// Magic: when 'user' is garbage collected, weakMap entry IS ALSO removed!
user = null; // user object eligible for GC → weakMap entry disappears too!
// No memory leak!
```

**Real-world use case — private data per instance:**
```js
// Pattern: use WeakMap to store private data per class instance
const _private = new WeakMap();

class BankAccount {
  constructor(owner, balance) {
    _private.set(this, {
      balance,
      transactions: []
    });
    this.owner = owner;
  }

  deposit(amount) {
    const data = _private.get(this);
    data.balance += amount;
    data.transactions.push({ type: "deposit", amount, date: Date.now() });
  }

  get balance() {
    return _private.get(this).balance;
  }

  getTransactions() {
    return [..._private.get(this).transactions]; // return copy
  }
}

const acc = new BankAccount("Alice", 1000);
acc.deposit(500);
acc.balance;      // 1500
acc._private;     // undefined — truly private!
// When 'acc' is GC'd, _private entry is also automatically removed
```

#### WeakSet:
```js
// WeakSet<Object>
// - Values MUST be objects
// - Values are weakly referenced
// - NOT iterable, no .size, no .forEach
// Use: tracking object membership without preventing GC

const weakSet = new WeakSet();

let div = document.createElement("div");
weakSet.add(div);
weakSet.has(div);  // true
weakSet.delete(div);

// If div is removed from DOM and all strong refs dropped:
div = null; // weakSet entry is automatically removed → no leak!
```

**WeakSet real-world use — lifecycle tracking:**
```js
const processedRequests = new WeakSet();

function processRequest(requestObj) {
  if (processedRequests.has(requestObj)) {
    throw new Error("Request already processed!"); // prevent double processing
  }
  processedRequests.add(requestObj);
  // ... process
}
// When requestObj goes out of scope → WeakSet doesn't hold it alive
```

| Feature | Map | WeakMap | Set | WeakSet |
|---|---|---|---|---|
| Key/value type | Any | Object keys | Any | Objects only |
| GC prevention | ✅ Prevents | ❌ Allows GC | ✅ Prevents | ❌ Allows GC |
| Iterable | ✅ Yes | ❌ No | ✅ Yes | ❌ No |
| `.size` prop | ✅ Yes | ❌ No | ✅ Yes | ❌ No |

---

### 8. What is Symbol in JavaScript?

**Definition:** `Symbol` is a **primitive data type** introduced in ES6. Every Symbol value is **guaranteed to be globally unique** — even if two Symbols have the same description, they are never equal. Symbols are immutable and can be used as **unique property keys** that won't conflict with other code.

```js
// Creating Symbols
const id1 = Symbol("id");
const id2 = Symbol("id"); // same description
id1 === id2; // false — every Symbol is unique!

typeof id1;   // "symbol"
id1.toString(); // "Symbol(id)"
id1.description; // "id"
```

**Symbol as object property keys:**
```js
// Symbols as keys avoid name collisions
const USER_ID = Symbol("userId");
const ADMIN_FLAG = Symbol("isAdmin");

const user = {
  name: "Alice",
  [USER_ID]: 12345,      // Symbol key
  [ADMIN_FLAG]: true,
};

user[USER_ID];   // 12345
user.name;       // "Alice"

// Symbols are hidden from normal enumeration!
Object.keys(user);             // ["name"] — no symbols!
Object.values(user);           // ["Alice"]
JSON.stringify(user);          // '{"name":"Alice"}' — symbols stripped!
for (const k in user) {}       // only "name"

// Only way to access symbols:
Object.getOwnPropertySymbols(user); // [Symbol(userId), Symbol(isAdmin)]
Reflect.ownKeys(user);             // ["name", Symbol(userId), Symbol(isAdmin)]
```

**Global Symbol Registry — `Symbol.for()`:**
```js
// Symbol.for() looks up or creates in the GLOBAL symbol registry
// Same key → same Symbol (unlike Symbol())
const s1 = Symbol.for("shared");
const s2 = Symbol.for("shared");
s1 === s2; // true  ← global registry, shared across modules/iframes!

Symbol.keyFor(s1); // "shared"  ← retrieve key from registry

// vs local Symbol:
const local = Symbol("shared");
symbol.keyFor(local); // undefined (not in global registry)
```

**Well-known Symbols — customizing built-in behavior:**
```js
// Symbol.iterator — make any object iterable (works with for...of)
class Range {
  constructor(start, end) {
    this.start = start;
    this.end = end;
  }

  [Symbol.iterator]() {
    let current = this.start;
    const end = this.end;
    return {
      next() {
        return current <= end
          ? { value: current++, done: false }
          : { value: undefined, done: true };
      }
    };
  }
}

const range = new Range(1, 5);
for (const n of range) console.log(n); // 1, 2, 3, 4, 5
[...range]; // [1, 2, 3, 4, 5]

// Symbol.toPrimitive — control type coercion
class Temperature {
  constructor(celsius) { this.celsius = celsius; }

  [Symbol.toPrimitive](hint) {
    if (hint === "number") return this.celsius;
    if (hint === "string") return `${this.celsius}°C`;
    return this.celsius; // default
  }
}

const temp = new Temperature(37);
+temp;          // 37 (number hint)
`${temp}`;      // "37°C" (string hint)
temp + 0;       // 37 (default hint)

// Symbol.hasInstance — customize instanceof behavior
class EvenNumber {
  static [Symbol.hasInstance](value) {
    return typeof value === "number" && value % 2 === 0;
  }
}
2  instanceof EvenNumber; // true
3  instanceof EvenNumber; // false
4  instanceof EvenNumber; // true

// Other well-known Symbols:
// Symbol.toStringTag  — custom Object.prototype.toString() output
// Symbol.species      — control what constructor is used in derived methods
// Symbol.asyncIterator — async iteration protocol
```

---

### 9. What are Iterators?

**Definition:** An **iterator** is an object that implements the **iterator protocol** by having a `next()` method that returns an object with `{ value, done }`. An **iterable** is any object that implements the **iterable protocol** by having a `[Symbol.iterator]()` method that returns an iterator.

```js
// The iterator protocol
const iterator = {
  next() {
    return { value: any, done: boolean };
  }
};
// done: false → still yielding values
// done: true  → exhausted, value is undefined (or a return value)
```

**Built-in iterables:** Arrays, Strings, Maps, Sets, NodeLists, arguments, generators
**What works with iterables:** `for...of`, spread `...`, destructuring, `Array.from()`, `Promise.all()`, `new Map([...])`, `new Set([...])`

**Custom iterator implementation:**
```js
// Infinite counter iterator
function createCounter(start = 0, step = 1) {
  let current = start;

  return {
    // The iterator itself
    next() {
      const value = current;
      current += step;
      return { value, done: false }; // never done
    },

    // Makes it also an iterable (self-referential)
    [Symbol.iterator]() { return this; }
  };
}

const counter = createCounter(0, 2);
counter.next(); // { value: 0, done: false }
counter.next(); // { value: 2, done: false }
counter.next(); // { value: 4, done: false }

// Use with for...of (must break manually for infinite iterators)
for (const n of createCounter(10)) {
  if (n > 20) break;
  console.log(n); // 10, 11, 12, ..., 20
}

// Finite iterable class
class LinkedList {
  constructor() { this.head = null; }

  add(val) {
    this.head = { val, next: this.head };
  }

  [Symbol.iterator]() {
    let current = this.head;
    return {
      next() {
        if (current) {
          const value = current.val;
          current = current.next;
          return { value, done: false };
        }
        return { value: undefined, done: true };
      }
    };
  }
}

const list = new LinkedList();
list.add(3); list.add(2); list.add(1);

for (const val of list) console.log(val); // 1, 2, 3
[...list]; // [1, 2, 3]
const [a, b] = list; // destructuring works!
```

**Lazy evaluation with iterators:**
```js
// Iterators are LAZY — they don't compute values until asked
// This enables infinite sequences and memory-efficient pipelines

function* naturals() {
  let n = 1;
  while (true) yield n++;
}

function* take(iterable, n) {
  let count = 0;
  for (const val of iterable) {
    if (count++ >= n) break;
    yield val;
  }
}

function* map(iterable, fn) {
  for (const val of iterable) yield fn(val);
}

function* filter(iterable, pred) {
  for (const val of iterable) if (pred(val)) yield val;
}

// Pipeline: first 5 even squares of natural numbers
const pipeline = take(
  filter(
    map(naturals(), n => n * n),    // 1, 4, 9, 16, 25, 36, ...
    n => n % 2 === 0                // 4, 16, 36, ...
  ),
  3                                 // take first 3
);

console.log([...pipeline]); // [4, 16, 36]
```

---

### 10. What are Generators?

**Definition:** A **generator function** (declared with `function*`) returns a **Generator object**, which is both an iterator and an iterable. Execution of the generator is **pausable** — it runs until it hits a `yield` expression, then pauses and returns the yielded value. It resumes where it left off on the next `.next()` call.

**This is unique — normal functions can't pause execution!**

```js
// Basic generator
function* greetings() {
  console.log("Start");
  yield "Hello";   // pause here, return "Hello"
  console.log("Middle");
  yield "World";   // pause here, return "World"
  console.log("End");
  return "Done";   // final return value (done: true)
}

const gen = greetings(); // Returns generator object — NOTHING executes yet!

gen.next(); // logs "Start", returns { value: "Hello", done: false }
gen.next(); // logs "Middle", returns { value: "World", done: false }
gen.next(); // logs "End",    returns { value: "Done",  done: true }
gen.next(); //                returns { value: undefined, done: true }
```

**Two-way communication with `.next(value)`:**
```js
function* calculator() {
  let result = 0;
  while (true) {
    const input = yield result; // pause, receive next .next(value)
    if (input === null) break;
    result += input;
  }
  return result;
}

const calc = calculator();
calc.next();    // start: { value: 0, done: false }
calc.next(10);  // add 10: { value: 10, done: false }
calc.next(20);  // add 20: { value: 30, done: false }
calc.next(5);   // add 5:  { value: 35, done: false }
calc.next(null); // stop:  { value: 35, done: true }
```

**Generators for async flow control (pre-async/await era):**
```js
// Generators were the original way to write async code like sync
// (libraries like co.js used this pattern)
function* fetchUserData(userId) {
  const user = yield fetch(`/api/users/${userId}`).then(r => r.json());
  const orders = yield fetch(`/api/orders/${user.id}`).then(r => r.json());
  return { user, orders };
}

// Runner that handles promises from generators
function run(generatorFn, ...args) {
  const gen = generatorFn(...args);

  function handle({ value, done }) {
    if (done) return Promise.resolve(value);
    return Promise.resolve(value).then(
      resolved => handle(gen.next(resolved)),
      rejected  => handle(gen.throw(rejected))
    );
  }

  return handle(gen.next());
}
// run(fetchUserData, 1).then(console.log);
// async/await superseded this pattern but generators are more powerful
```

**Infinite lazy sequences:**
```js
function* fibonacci() {
  let [a, b] = [0, 1];
  while (true) {
    yield a;
    [a, b] = [b, a + b];
  }
}

function* take(gen, n) {
  for (let i = 0; i < n; i++) {
    const { value, done } = gen.next();
    if (done) break;
    yield value;
  }
}

[...take(fibonacci(), 10)]; // [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]

// Generators for unique ID generation
function* idGenerator(prefix = "id") {
  let i = 1;
  while (true) {
    yield `${prefix}_${i++}`;
  }
}

const ids = idGenerator("user");
ids.next().value; // "user_1"
ids.next().value; // "user_2"
ids.next().value; // "user_3"
```

**`yield*` — delegate to another iterable/generator:**
```js
function* inner() {
  yield "a";
  yield "b";
}

function* outer() {
  yield 1;
  yield* inner();       // delegate to inner generator
  yield* [10, 20, 30];  // delegate to array iterator
  yield 2;
}

[...outer()]; // [1, "a", "b", 10, 20, 30, 2]
```

---## ⚡ Section 2: Asynchronous JavaScript (Deep Dive)

---

### 11. Explain Microtask Queue vs Macrotask Queue

**Definition:** JavaScript's concurrency model uses a single-threaded event loop with multiple queues of different priorities. Understanding these queues is critical for predicting execution order.

**The complete execution model:**
```
                    ┌────────────────────────┐
                    │      Call Stack        │  ← synchronous execution
                    └────────────┬───────────┘
                                 │ empty?
                    ┌────────────▼───────────┐
                    │    Microtask Queue     │  ← DRAIN COMPLETELY first
                    │  • Promise .then/catch │     (all microtasks run before
                    │  • queueMicrotask()    │      any macrotask can start)
                    │  • MutationObserver    │
                    └────────────┬───────────┘
                                 │ empty?
                    ┌────────────▼───────────┐
                    │ Render Pipeline        │  ← only in browsers, not Node.js
                    │  • requestAnimationFrame│
                    │  • layout / paint      │
                    └────────────┬───────────┘
                                 │
                    ┌────────────▼───────────┐
                    │    Macrotask Queue     │  ← ONE task per loop iteration
                    │  • setTimeout()        │
                    │  • setInterval()       │
                    │  • setImmediate()(Node)│
                    │  • I/O callbacks       │
                    │  • MessageChannel      │
                    └────────────────────────┘
```

**Priority: Call Stack > Microtasks > rendering > Macrotasks**

**Detailed execution tracing:**
```js
console.log("1 - sync start");

setTimeout(() => console.log("2 - setTimeout (macrotask)"), 0);

Promise.resolve()
  .then(() => {
    console.log("3 - Promise.then (microtask 1)");
    // Adding microtask from within microtask — runs BEFORE next macrotask!
    return Promise.resolve();
  })
  .then(() => console.log("4 - Promise.then (microtask 2)"));

queueMicrotask(() => console.log("5 - queueMicrotask (microtask 3)"));

Promise.resolve().then(() => console.log("6 - Promise.then (microtask 4)"));

console.log("7 - sync end");

// Execution order:
// 1 - sync start       (call stack - sync)
// 7 - sync end         (call stack - sync)
// 3 - Promise.then     (microtask queue flush - first pass)
// 5 - queueMicrotask   (microtask queue flush - first pass)
// 6 - Promise.then     (microtask queue flush - first pass)
// 4 - Promise.then     (microtask queue flush - second pass, added during flush)
// 2 - setTimeout       (macrotask - only after ALL microtasks done)
```

**Critical rule:** After every macrotask completes, and after synchronous code finishes, the microtask queue is **completely drained** — even if new microtasks are added during draining!

```js
// This could STARVE the macrotask queue (and block rendering)!
function recursiveMicrotask() {
  Promise.resolve().then(() => {
    console.log("microtask");
    recursiveMicrotask(); // schedules another microtask!
    // Macrotasks NEVER get a turn → setTimeout never fires!
  });
}
recursiveMicrotask();
setTimeout(() => console.log("never runs"), 100); // starved!
```

---

### 12. How Does Promise Chaining Work?

**Definition:** Promise chaining exploits the fact that `.then()`, `.catch()`, and `.finally()` **always return a new Promise**. This new promise resolves with the return value of the handler, enabling sequential async operations without nesting.

**The mechanics:**
```js
// Each .then() creates a NEW Promise
const p1 = Promise.resolve(1);

const p2 = p1.then(val => val + 1);  // p2 resolves with 2
const p3 = p2.then(val => val * 3);  // p3 resolves with 6
const p4 = p3.then(val => val - 1);  // p4 resolves with 5

p4.then(console.log); // 5
```

**Chain with async operations:**
```js
// ✅ Clean chaining — each handler receives previous handler's return value
fetch("/api/users/1")
  .then(res => {
    if (!res.ok) throw new Error(`HTTP ${res.status}`); // throw → goes to catch
    return res.json();                 // returning a Promise = awaiting it!
  })
  .then(user => {
    console.log("User:", user.name);
    return fetch(`/api/orders/${user.id}`); // return another Promise
    // chain waits for this fetch to resolve!
  })
  .then(res => res.json())
  .then(orders => {
    console.log("Orders:", orders.length);
    return orders.filter(o => o.status === "active");
  })
  .then(activeOrders => displayOrders(activeOrders))
  .catch(err => {
    // 🎯 ONE catch handles errors from ANY point in the chain!
    console.error("Something went wrong:", err.message);
    return []; // recovering — chain continues with []
  })
  .finally(() => {
    hideLoadingSpinner(); // ALWAYS runs, regardless of success/failure
  });
```

**Common trap — broken chains:**
```js
// ❌ WRONG — creates parallel chains, errors not propagated to outer chain
fetch("/api/data")
  .then(res => {
    res.json().then(data => displayData(data)); // inner chain not returned!
    // This returns undefined → outer chain resolves with undefined
  });

// ✅ CORRECT — return the inner promise to maintain chain
fetch("/api/data")
  .then(res => res.json())       // return the Promise
  .then(data => displayData(data));
```

**`.catch()` is `.then(undefined, onRejected)`:**
```js
// Placed at end — catches all earlier rejections
// ✅ Best practice for error handling
promise
  .then(a)
  .then(b)
  .then(c)
  .catch(handleError);

// Placed in middle — catches up to that point, allows chain to continue
promise
  .then(a)
  .catch(err => { console.warn(err); return "default"; }) // recover
  .then(b);  // receives "default" if a failed
```

---

### 13. What is `Promise.all` vs `Promise.allSettled`?

**Comparison of all Promise combinators:**

```js
const p1 = Promise.resolve("Alice");
const p2 = Promise.reject(new Error("Not found"));
const p3 = Promise.resolve(42);
```

#### `Promise.all()` — all must succeed:
```js
// Resolves: array of fulfilled values (order preserved, regardless of completion order)
// Rejects:  immediately when FIRST promise rejects (fail-fast!)
// Use case: parallel operations that ALL must succeed

const [user, product] = await Promise.all([
  User.findById(id),        // must succeed
  Product.findById(pid)     // must succeed
]);

// If either fails, the whole thing fails:
try {
  await Promise.all([p1, p2, p3]);          // p2 rejects → immediately rejects!
} catch (err) {
  console.log(err.message); // "Not found"   (p1 and p3 results discarded)
}

// Practical: fetch dashboard data — need all or none
const [users, revenue, notifications] = await Promise.all([
  fetchUsers(),
  fetchRevenue(),
  fetchNotifications()
]);
// If notifications fail → we have partial data, but Promise.all throws
```

#### `Promise.allSettled()` — waits for all, whatever the result:
```js
// ALWAYS resolves (never rejects) with array of result objects:
// { status: "fulfilled", value: V } or { status: "rejected", reason: E }
// Use case: independent operations where partial success is acceptable

const results = await Promise.allSettled([p1, p2, p3]);
// [
//   { status: "fulfilled", value: "Alice" },
//   { status: "rejected",  reason: Error("Not found") },
//   { status: "fulfilled", value: 42 }
// ]

results.forEach(result => {
  if (result.status === "fulfilled") {
    console.log("Success:", result.value);
  } else {
    console.error("Failed:", result.reason);
  }
});

// Practical: send notifications to multiple users (partial failure OK)
const notifications = users.map(user => sendEmail(user.email));
const results = await Promise.allSettled(notifications);

const failed = results.filter(r => r.status === "rejected");
console.log(`${failed.length} emails failed to send`);
```

#### `Promise.race()` — first to settle wins:
```js
// Resolves/rejects with FIRST settled promise's outcome
// Use case: timeout patterns, redundant requests

const withTimeout = (promise, ms) =>
  Promise.race([
    promise,
    new Promise((_, reject) =>
      setTimeout(() => reject(new Error(`Timeout after ${ms}ms`)), ms)
    )
  ]);

const user = await withTimeout(fetchUser(id), 3000);
// Either resolves with user, or rejects with timeout error
```

#### `Promise.any()` — first to FULFILL wins (ES2021):
```js
// Resolves: first fulfilled promise's value
// Rejects: AggregateError if ALL promises reject
// Use case: try multiple sources, take whichever responds first

const data = await Promise.any([
  fetchFromCDN1(url),
  fetchFromCDN2(url),
  fetchFromCDN3(url),
]);
// Gets data from whichever CDN responds first

try {
  await Promise.any([
    Promise.reject(new Error("CDN1 down")),
    Promise.reject(new Error("CDN2 down")),
  ]);
} catch (e) {
  console.log(e instanceof AggregateError); // true
  console.log(e.errors); // [Error("CDN1 down"), Error("CDN2 down")]
}
```

| Combinator | Resolves | Rejects | Order preserved |
|---|---|---|---|
| `Promise.all` | All fulfilled | First rejection | ✅ |
| `Promise.allSettled` | Always (all settled) | Never | ✅ |
| `Promise.race` | First settled | First rejection | ❌ |
| `Promise.any` | First fulfilled | All rejected (AggregateError) | ❌ |

---

### 14. How Do You Run Async Operations in Parallel?

**Definition:** Parallel execution in Node.js means **starting multiple async operations simultaneously** without waiting for each to finish before starting the next. Since Node.js is single-threaded, "parallel" means the I/O operations (DB, HTTP, file) run concurrently in the background via the event loop.

```js
// ❌ SEQUENTIAL — each waits for the previous (total = sum of all durations)
async function sequential() {
  const user    = await User.findById(id);       // 100ms
  const orders  = await Order.find({ userId: id }); // 200ms
  const profile = await Profile.findOne({ userId: id }); // 150ms
  // Total: 450ms   ← slow!
  return { user, orders, profile };
}

// ✅ PARALLEL — all start simultaneously (total = max of durations)
async function parallel() {
  const [user, orders, profile] = await Promise.all([
    User.findById(id),                    // \
    Order.find({ userId: id }),           //  ├── all run at same time!
    Profile.findOne({ userId: id }),      // /
  ]);
  // Total: ~200ms   ← fast!
  return { user, orders, profile };
}
```

**Manual parallel start (avoid `await` in loops):**
```js
// ❌ Sequential (wrong) — each loop iteration waits
async function processAllUsers(userIds) {
  const results = [];
  for (const id of userIds) {
    results.push(await processUser(id)); // awaits each one before next!
  }
  return results;
}

// ✅ Parallel — start all then wait
async function processAllUsers(userIds) {
  const promises = userIds.map(id => processUser(id)); // start all!
  return Promise.all(promises);                         // wait for all
}

// ✅ With concurrency control (avoid overwhelming DB/API)
async function processWithConcurrency(items, fn, concurrency = 5) {
  const results = [];
  for (let i = 0; i < items.length; i += concurrency) {
    const batch = items.slice(i, i + concurrency);
    const batchResults = await Promise.all(batch.map(fn));
    results.push(...batchResults);
  }
  return results;
}

// Process 100 users in batches of 5
await processWithConcurrency(userIds, processUser, 5);
```

**Mixed sequential and parallel:**
```js
async function loadDashboard(userId) {
  // Step 1: MUST get user first (sequential)
  const user = await User.findById(userId);

  // Step 2: PARALLEL — user is available for all
  const [orders, followers, posts] = await Promise.all([
    Order.find({ userId: user._id }),
    Follower.count({ followedId: user._id }),
    Post.find({ authorId: user._id }).sort({ date: -1 }).limit(5)
  ]);

  // Step 3: Sequential — needs orders from step 2
  const topOrder = await Order.findById(orders[0]?._id).populate("items");

  return { user, orders, followers, posts, topOrder };
}
```

---

### 15. How Do You Handle Errors in async/await?

**Definition:** Error handling in async/await is done primarily with `try/catch/finally` blocks, which catch synchronous throws, Promise rejections, and runtime errors uniformly. Proper error handling is essential for production Node.js applications.

```js
// 1. Basic try/catch
async function fetchUser(id) {
  try {
    const user = await User.findById(id);
    if (!user) throw new Error("User not found"); // manual throw
    return user;
  } catch (err) {
    // Catches: rejected promises, thrown errors, runtime errors
    console.error(err.message);
    throw err; // re-throw to propagate up
  } finally {
    closeConnection(); // always runs
  }
}

// 2. Type-specific error handling
async function processPayment(orderId) {
  try {
    const order = await Order.findById(orderId);
    await chargeCard(order.cardToken, order.total);
    await Order.updateOne({ _id: orderId }, { status: "paid" });
  } catch (err) {
    if (err.name === "ValidationError") {
      return { success: false, error: "Invalid payment data" };
    }
    if (err.code === "CARD_DECLINED") {
      await notifyUser(order.userId, "Payment declined");
      return { success: false, error: "Card declined" };
    }
    // Unknown error — re-throw
    throw err;
  }
}

// 3. Result wrapper pattern (no throw propagation)
async function safeAsync(fn, ...args) {
  try {
    const data = await fn(...args);
    return { data, error: null };
  } catch (error) {
    return { data: null, error };
  }
}

const { data: user, error } = await safeAsync(fetchUser, userId);
if (error) {
  console.error("Failed to fetch user:", error.message);
} else {
  console.log(user.name);
}

// 4. Wrapping Express route handlers
const asyncHandler = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next); // passes error to Express error handler
};

app.get("/users/:id", asyncHandler(async (req, res) => {
  const user = await User.findById(req.params.id);
  if (!user) return res.status(404).json({ error: "Not found" });
  res.json(user);
  // Any thrown error → goes to Express global error handler via next(err)
}));

// 5. Global unhandled rejection handler (last line of defense)
process.on("unhandledRejection", (reason, promise) => {
  console.error("Unhandled Rejection:", reason);
  // Log to error tracking service (Sentry, etc.)
  // Decide: crash and restart, or continue?
  process.exit(1); // recommended: crash fast, let process manager restart
});
```

---

### 16. What is Event Loop Starvation?

**Definition:** Event loop starvation occurs when **high-priority tasks continuously prevent lower-priority tasks from executing**. In JavaScript, this mainly happens when the microtask queue never empties (preventing macrotasks from running) or when synchronous code / long-running operations block the event loop entirely.

```js
// ❌ Microtask starvation — macrotasks never run
function starveMacrotasks() {
  Promise.resolve().then(function recurse() {
    // Each .then schedules a new microtask before macrotask queue gets turn
    Promise.resolve().then(recurse);
  });
}
starveMacrotasks();
setTimeout(() => console.log("I never run!"), 0); // starved!
// Microtask queue is never empty → event loop never picks up setTimeout

// ❌ CPU-blocking synchronous code — blocks EVERYTHING
function blockEventLoop(ms) {
  const start = Date.now();
  while (Date.now() - start < ms) {
    // spinning — blocks all async callbacks, I/O, timers!
  }
}
blockEventLoop(5000); // server is completely frozen for 5 seconds!

// ✅ Breaking long computation into chunks (yield to event loop)
async function processLargeDataset(items) {
  const CHUNK_SIZE = 1000;
  const results = [];

  for (let i = 0; i < items.length; i += CHUNK_SIZE) {
    const chunk = items.slice(i, i + CHUNK_SIZE);
    results.push(...chunk.map(heavyComputation));

    // Yield to event loop between chunks!
    // Allows I/O callbacks, timers, and other requests to process
    await new Promise(resolve => setImmediate(resolve));
  }
  return results;
}

// ✅ Better: Worker Threads for truly CPU-intensive work
const { Worker, isMainThread, parentPort } = require("worker_threads");
// Offload heavy computation to a separate thread
// Main thread event loop remains unblocked
```

---

### 17. How Does Node.js Handle Concurrency?

**Definition:** Node.js achieves concurrency on a single thread through its **event loop architecture** backed by **libuv** — a C library that provides the actual I/O operations, a thread pool, and platform abstractions.

**The architecture:**
```
Node.js Process
├── V8 Engine (JavaScript execution — single thread)
│   └── Call Stack
│
├── libuv
│   ├── Event Loop (orchestrates everything)
│   │   ├── Phase 1: timers (setTimeout, setInterval callbacks)
│   │   ├── Phase 2: pending callbacks (I/O errors)
│   │   ├── Phase 3: idle, prepare (internal)
│   │   ├── Phase 4: poll (wait for I/O events ← most time spent here)
│   │   ├── Phase 5: check (setImmediate callbacks)
│   │   └── Phase 6: close callbacks (socket.on("close"))
│   │
│   └── Thread Pool (default: 4 threads)
│       ├── File system (fs) operations
│       ├── DNS lookups
│       ├── crypto operations (heavy ones)
│       └── User-defined (via Worker Threads API)
│
├── Network I/O (epoll/kqueue/IOCP — OS-level, no thread pool!)
│   └── TCP/UDP sockets, HTTP requests
│
└── Worker Threads (true parallelism for CPU-bound code)
    ├── Thread 1 (image processing)
    ├── Thread 2 (ML inference)
    └── Thread 3 (heavy calculations)
```

```js
// Network I/O — handled by OS, NO thread pool needed
// All happen "truly" concurrently at OS level:
http.get("https://api1.com", cb1);  // OS handles via non-blocking sockets
http.get("https://api2.com", cb2);  // all running simultaneously!
http.get("https://api3.com", cb3);

// File system — uses libuv thread pool (4 threads by default)
// Increase thread pool for heavily I/O-bound apps:
process.env.UV_THREADPOOL_SIZE = "8"; // must be set before requiring modules!

// Worker Threads — true parallel JS for CPU-bound work
const { Worker } = require("worker_threads");

function runImageResize(imagePath) {
  return new Promise((resolve, reject) => {
    const worker = new Worker("./imageWorker.js", {
      workerData: { path: imagePath }
    });
    worker.on("message", resolve);
    worker.on("error", reject);
  });
}

// Multiple resizes run in TRUE parallel (different OS threads)
const [img1, img2, img3] = await Promise.all([
  runImageResize("photo1.jpg"),
  runImageResize("photo2.jpg"),
  runImageResize("photo3.jpg"),
]);
```

---

### 18. What is `setImmediate`?

**Definition:** `setImmediate()` is a Node.js-specific function that schedules a callback to execute in the **"check" phase** of the event loop — after the current I/O events have been processed but **before** any timers (`setTimeout`), if both are scheduled in the same I/O cycle.

```js
// setImmediate vs setTimeout(fn, 0) — ORDER IS NOT GUARANTEED at top level!
setImmediate(() => console.log("setImmediate"));
setTimeout(() => console.log("setTimeout 0"), 0);

// Output is non-deterministic at top level:
// Sometimes: setImmediate → setTimeout 0
// Sometimes: setTimeout 0 → setImmediate
// (depends on OS timer granularity)

// INSIDE an I/O callback: setImmediate ALWAYS beats setTimeout(fn, 0)
const fs = require("fs");
fs.readFile(__filename, () => {
  setImmediate(() => console.log("setImmediate (inside I/O)"));
  setTimeout(() => console.log("setTimeout 0 (inside I/O)"), 0);
  // Always: setImmediate first, then setTimeout
  // Because: we're already in poll phase, next is check (setImmediate)
});

// Use case: allow I/O events to be processed between iterations
function processHeavyTask(items, callback) {
  let index = 0;

  function processNext() {
    if (index >= items.length) return callback(null, "done");

    processItem(items[index++]);

    // Yield to I/O — allow pending requests to be handled
    setImmediate(processNext);
  }

  processNext();
}
```

---

### 19. What is `process.nextTick()`?

**Definition:** `process.nextTick()` schedules a callback to execute **before the next iteration of the event loop starts** — it doesn't wait for I/O, timers, or any phase. It runs **immediately after the current operation completes**, even before Promises (microtasks). It's part of the "nextTickQueue" which is checked before the microtask queue.

```
Call Stack Empties
       ↓
nextTick queue (drains completely)  ← process.nextTick
       ↓
microtask queue (drains completely) ← Promise.then
       ↓
Next Event Loop Phase (timers/I/O/check/...)
```

```js
process.nextTick(() => console.log("nextTick 1"));
Promise.resolve().then(() => console.log("Promise 1"));
process.nextTick(() => console.log("nextTick 2"));
Promise.resolve().then(() => console.log("Promise 2"));

// Output:
// nextTick 1
// nextTick 2   ← nextTick queue fully drained first
// Promise 1
// Promise 2    ← Promise (microtask) queue drained next

// Use case: emit events asynchronously (allow listener registration first)
class EventualEmitter extends EventEmitter {
  start() {
    // If we emit synchronously, listeners haven't been attached yet!
    // nextTick defers until current stack frame is done
    process.nextTick(() => this.emit("start", { time: Date.now() }));
    return this;
  }
}

const emitter = new EventualEmitter();
emitter.start(); // starts scheduling 'start' event
emitter.on("start", (data) => console.log("Started!", data)); // listener registered
// Works! nextTick hasn't fired yet when .on() is called
```

---

### 20. Difference Between `process.nextTick` vs `setImmediate`?

```
Order within a single event loop iteration:

1. [ Synchronous code runs ]
2. [ process.nextTick queue drains ]  ← nextTick
3. [ Promise microtask queue drains ] ← Promise.then
4. [ Event loop phase executes ]
   ├── timers phase (setTimeout, setInterval)
   ├── poll phase (I/O callbacks)
   └── check phase (setImmediate)          ← setImmediate
5. Repeat
```

| Feature | `process.nextTick()` | `setImmediate()` |
|---|---|---|
| Queue | nextTickQueue (before microtasks) | check phase of event loop |
| Timing | End of current operation | Next check phase |
| Relative to Promise | **Before** Promise.then | **After** Promise.then |
| Relative to each other | **Before** setImmediate | After nextTick |
| Risk of starvation | ✅ High (recursive nextTick starves I/O!) | ✅ Low (one pass per loop) |
| Node.js only | ✅ Yes | ✅ Yes |

```js
setImmediate(() => console.log("setImmediate"));
process.nextTick(() => console.log("nextTick"));
Promise.resolve().then(() => console.log("Promise"));

// Always output:
// nextTick   (runs first — before everything)
// Promise    (microtask — after nextTick, before event loop phases)
// setImmediate (event loop check phase)

// ⚠️ Recursive nextTick = STARVATION (worse than recursive Promises!)
function recursiveNextTick() {
  process.nextTick(() => {
    console.log("tick");
    recursiveNextTick(); // schedules ANOTHER nextTick immediately!
  });
}
recursiveNextTick();
// I/O callbacks, timers, and setImmediate NEVER run!
// process.nextTick recursion is more dangerous than Promise recursion

// ✅ Recommendation (Node.js docs):
// Use setImmediate in most cases — it's more predictable
// Use process.nextTick only when you need to run code before any I/O
//   (e.g., error propagation, ensuring consistent async/sync behavior)
```

---

## 🧠 Section 3: Closures & Functional Programming

---

### 21. What is a Closure? (Deep Dive)

**Definition:** A **closure** is the combination of a function and the **lexical environment** within which that function was declared. This environment consists of any local variables that were in-scope at the time the closure was created.

**How it works (Internally):**
When a function is defined, it stores a reference to its outer lexical environment in an internal property (often called `[[Environment]]`). Even after the outer function finishes execution and its execution context is popped off the stack, the variables in the outer environment remain in memory if they are still referenced by the inner function.

```javascript
function makeCounter() {
  let count = 0; // Lexical environment of makeCounter

  return function() {
    // This inner function is a closure
    return ++count; 
  };
}

const counter = makeCounter();
console.log(counter()); // 1
console.log(counter()); // 2
```

**Real-world use cases:**
1.  **Data Privacy / Encapsulation:** Creating "private" variables that can only be accessed via specific methods.
2.  **Function Factories:** Creating specialized versions of functions.
3.  **Partial Application/Currying:** Pre-filling arguments of a function.
4.  **Emulating Private Methods:** Useful in patterns like the Module pattern.

**Memory Considerations:** Closures can lead to higher memory usage because the outer scope variables are not garbage collected as long as the closure exists.

---

### 22. What is Currying?

**Definition:** **Currying** is a functional programming technique where a function with multiple arguments is transformed into a **sequence of nesting functions**, each taking a single argument.

**Example:**
```javascript
// Normal function
function add(a, b) {
  return a + b;
}

// Curried version
function curriedAdd(a) {
  return function(b) {
    return a + b;
  };
}

const addFive = curriedAdd(5); // Specialized function
console.log(addFive(3)); // 8
console.log(curriedAdd(5)(10)); // 15
```

**Why use it?**
-   **Partial Application:** You can create specialized functions by fixing some arguments.
-   **Code Reusability:** It helps in building generic functions that can be easily customized.
-   **Composition:** Curried functions are easier to compose with other functions.

---

### 23. What is Partial Application?

**Definition:** **Partial application** is the process of fixing a number of arguments to a function, producing another function of smaller arity (fewer arguments).

**Difference from Currying:**
-   **Currying** always transforms a function into a chain of unary (single-argument) functions.
-   **Partial application** takes a function with N arguments and returns a function with N-K arguments (where K is the number of pre-filled arguments).

**Example:**
```javascript
const multiply = (a, b, c) => a * b * c;

// Partial application using .bind()
const multiplyByTwo = multiply.bind(null, 2); 
console.log(multiplyByTwo(3, 4)); // 24 (2 * 3 * 4)

// Manual implementation
const partial = (fn, ...args) => (...rest) => fn(...args, ...rest);
const doubleAndTriple = partial(multiply, 2, 3);
console.log(doubleAndTriple(10)); // 60 (2 * 3 * 10)
```

---

### 24. What is Function Composition?

**Definition:** **Composition** is the act of combining two or more functions to produce a new function. In mathematics, it's often represented as `f(g(x))`.

**Example:**
```javascript
const toUpperCase = x => x.toUpperCase();
const exclaim = x => `${x}!`;
const emphasis = x => `**${x}**`;

// Manual composition (Right to Left)
const formatText = x => emphasis(exclaim(toUpperCase(x)));
console.log(formatText("hello")); // **HELLO!**

// Generic compose utility (Right to Left)
const compose = (...fns) => x => fns.reduceRight((v, f) => f(v), x);

const shout = compose(emphasis, exclaim, toUpperCase);
console.log(shout("javascript")); // **JAVASCRIPT!**
```

**Pipe vs Compose:**
-   **Compose:** Evaluates right-to-left (standard mathematical way).
-   **Pipe:** Evaluates left-to-right (more intuitive for readability).

---

### 25. What is a Pure Function?

**Definition:** A **pure function** is a function that satisfies two conditions:
1.  **Deterministic:** It always returns the same output for the same input.
2.  **No Side Effects:** It does not modify any state outside its scope (e.g., doesn't change global variables, doesn't print to console, doesn't make network requests).

**Example:**
```javascript
// Pure
const add = (a, b) => a + b; 

// Impure (Side effect: console.log)
const addAndLog = (a, b) => {
  console.log(a + b);
  return a + b;
};

// Impure (Deterministic failure: relies on external state)
let secret = 10;
const addSecret = (a) => a + secret; 
```

**Benefits:**
-   **Ease of Testing:** No mocking required for external state.
-   **Predictability:** No "hidden" bugs from state mutations.
-   **Memoization:** Pure functions can easily be cached.
-   **Reasoning:** Makes it easier to understand code flow.

---

### 26. What is Immutability?

**Definition:** **Immutability** means that once a piece of data is created, it **cannot be changed**. Instead of modifying the existing data, you create a new copy with the desired changes.

**Why is it important?**
-   **Predictability:** You know your data won't "vanish" or change under your feet.
-   **Concurrency:** Safer in multi-threaded environments (though JS is single-threaded, it helps with async state).
-   **Tracking Changes:** Makes it trivial to check if something changed (referential equality check `old === new`).

**How to achieve it in JS:**
```javascript
// ❌ Mutation
const user = { name: "Alice", age: 25 };
user.age = 26; 

// ✅ Immutability (Spread operator)
const updatedUser = { ...user, age: 26 };

// ✅ Immutability (Arrays)
const list = [1, 2, 3];
const newList = [...list, 4]; // add
const filteredList = list.filter(x => x !== 2); // remove

// Object.freeze (shallow freeze)
const config = Object.freeze({ theme: "dark" });
config.theme = "light"; // Error in strict mode, ignored otherwise
```

---

### 27. What is Memoization?

**Definition:** **Memoization** is an optimization technique used to speed up computer programs by **storing the results of expensive function calls** and returning the cached result when the same inputs occur again.

**Example implementation:**
```javascript
function memoize(fn) {
  const cache = new Map();
  return function(...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) return cache.get(key);
    
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

const square = memoize(x => {
  console.log("Computing...");
  return x * x;
});

console.log(square(5)); // Computing... 25
console.log(square(5)); // 25 (from cache)
```

**When to use:**
-   **Pure functions:** Only pure functions can be effectively memoized.
-   **Expensive computations:** Fibonacci, factorial, heavy data processing.
-   **High frequency:** Functions called often with the same arguments.

---

### 28. What are Higher-Order Functions (HOF)?

**Definition:** A **Higher-Order Function** is a function that does at least one of the following:
1.  Takes one or more functions as arguments (e.g., `callback`).
2.  Returns a function as its result.

**Examples:**
-   **Built-in:** `map`, `filter`, `reduce`, `addEventListener`.
-   **Custom:** `memoize`, `debounce`, `throttle`, `compose`.

```javascript
// Takes a function
const repeat = (n, action) => {
  for (let i = 0; i < n; i++) action(i);
};

// Returns a function
const multiplyBy = (factor) => (num) => num * factor;
```

---

### 29. What is the difference between Imperative and Declarative programming?

| Feature | Imperative | Declarative |
|---|---|---|
| **Focus** | How to do it (Steps) | What to do (Result) |
| **Control Flow** | Explicit (loops, if/else) | Implied (map, filter, reduce) |
| **State** | Managed explicitly | Often avoided or immutable |
| **Readability** | Can be verbose and hard to follow | Concise and readable |

**Example (Filtering even numbers):**
```javascript
const numbers = [1, 2, 3, 4, 5];

// Imperative
const evenInc = [];
for (let i = 0; i < numbers.length; i++) {
  if (numbers[i] % 2 === 0) {
    evenInc.push(numbers[i]);
  }
}

// Declarative
const evenDec = numbers.filter(n => n % 2 === 0);
```

---

### 30. What is Recursion and how to optimize it?

**Definition:** **Recursion** is a technique where a function calls itself to solve a smaller instance of the same problem.

**Components:**
1.  **Base Case:** The condition that stops the recursion.
2.  **Recursive Step:** Reducing the problem and calling itself.

**Tail Call Optimization (TCO):**
TCO is an optimization where the compiler doesn't add a new stack frame if the recursive call is the last action in the function. (Note: Only supported in Safari/Webkit among major browsers as of ES6).

**Example (Factorial):**
```javascript
// Standard recursion (Not TCO)
function fact(n) {
  if (n === 0) return 1;
  return n * fact(n - 1); // Not the last action (multiplication is last)
}

// Tail-recursive (TCO candidate)
function factTCO(n, acc = 1) {
  if (n === 0) return acc;
  return factTCO(n - 1, n * acc); // Recursive call is the last action
}
```

---

## 🏗️ Section 4: Design Patterns & Architecture

---

### 31. What is the Module Pattern?

**Definition:** The **Module Pattern** is used to mimic the concept of classes in a way that we can include both public and private methods and variables inside a single object, shielding particular parts from the global scope.

**Implementation (using Closure):**
```javascript
const UserModule = (function() {
  // Private variables and methods
  let _users = []; 

  function _validate(user) {
    return user.name && user.email;
  }

  // Public API
  return {
    addUser(user) {
      if (_validate(user)) {
        _users.push(user);
        console.log("User added!");
      }
    },
    getUsers() {
      return [..._users]; // Return a copy to keep original array private
    }
  };
})();

UserModule.addUser({ name: "Alice", email: "alice@example.com" });
console.log(UserModule.getUsers()); // [{ name: "Alice", ... }]
console.log(UserModule._users); // undefined (Private!)
```

**Modern Alternative:** ES Modules (`import`/`export`) are the standard way to achieve modularity in modern JS.

---

### 32. What is the Singleton Pattern?

**Definition:** The **Singleton Pattern** restricts the instantiation of a class to **one single instance**. This is useful when exactly one object is needed to coordinate actions across the system (e.g., a Database connection pool or a global State Manager).

**Implementation:**
```javascript
class Database {
  constructor() {
    if (Database.instance) {
      return Database.instance;
    }
    this.connection = "Connected to DB";
    Database.instance = this;
  }
}

const db1 = new Database();
const db2 = new Database();

console.log(db1 === db2); // true (Same instance)
```

**Alternative (using Modules):** Because ES modules are cached after the first import, simply exporting an object or an instance of a class from a module creates a singleton.

---

### 33. What is the Observer Pattern?

**Definition:** A design pattern where an object (the **Subject**) maintains a list of its dependents (**Observers**) and notifies them automatically of any state changes, usually by calling one of their methods.

**Example:**
```javascript
class Subject {
  constructor() {
    this.observers = [];
  }

  subscribe(fn) {
    this.observers.push(fn);
  }

  unsubscribe(fn) {
    this.observers = this.observers.filter(sub => sub !== fn);
  }

  notify(data) {
    this.observers.forEach(fn => fn(data));
  }
}

const newsAgency = new Subject();

const subscriber1 = (news) => console.log(`Sub 1 received: ${news}`);
const subscriber2 = (news) => console.log(`Sub 2 received: ${news}`);

newsAgency.subscribe(subscriber1);
newsAgency.subscribe(subscriber2);

newsAgency.notify("Breaking News: JS is awesome!");
```

**Real-world usage:** `addEventListener`, Redux store subscriptions, RxJS.

---

### 34. What is the Factory Pattern?

**Definition:** A **Factory** is a function or class that creates other objects. It's used when the object creation logic is complex or depends on certain conditions.

**Example:**
```javascript
class Developer {
  constructor(name) {
    this.name = name;
    this.type = "Developer";
  }
}

class Manager {
  constructor(name) {
    this.name = name;
    this.type = "Manager";
  }
}

class EmployeeFactory {
  create(name, type) {
    switch(type) {
      case 1: return new Developer(name);
      case 2: return new Manager(name);
      default: return null;
    }
  }
}

const factory = new EmployeeFactory();
const dev = factory.create("Alice", 1);
const mgr = factory.create("Bob", 2);
```

---

### 35. What is the Proxy Pattern?

**Definition:** The **Proxy Pattern** provides a surrogate or placeholder for another object to control access to it. In JS, the `Proxy` object allows you to intercept and redefine fundamental operations (e.g., property lookup, assignment, enumeration, function invocation).

**Example:**
```javascript
const user = { name: "Alice", age: 25 };

const userProxy = new Proxy(user, {
  get(target, prop) {
    console.log(`Getting property: ${prop}`);
    return target[prop].toUpperCase();
  },
  set(target, prop, value) {
    if (prop === 'age' && typeof value !== 'number') {
      throw new TypeError("Age must be a number");
    }
    console.log(`Setting ${prop} to ${value}`);
    target[prop] = value;
    return true; // Success
  }
});

userProxy.name; // Getting property: name -> "ALICE"
userProxy.age = 30; // Setting age to 30
// userProxy.age = "thirty"; // Throws TypeError
```

**Use cases:** Validation, Logging/Profiling, Debugging, Revocable references.

---

### 36. What is the Mediator Pattern?

**Definition:** A behavioral design pattern that reduces chaotic dependencies between objects. The pattern restricts direct communications between the objects and forces them to collaborate only via a **mediator object**.

**Example:**
```javascript
class ChatRoom {
  // The Mediator
  showMessage(user, message) {
    const time = new Date().toLocaleTimeString();
    const sender = user.getName();
    console.log(`${time} [${sender}]: ${message}`);
  }
}

class User {
  constructor(name, chatRoom) {
    this.name = name;
    this.chatRoom = chatRoom;
  }
  getName() { return this.name; }
  send(message) {
    this.chatRoom.showMessage(this, message);
  }
}

const chat = new ChatRoom();
const alice = new User("Alice", chat);
const bob = new User("Bob", chat);

alice.send("Hi Bob!");
bob.send("Hey Alice!");
```

---

### 37. What is Command Pattern?

**Definition:** A pattern that turns a request into a stand-alone object that contains all information about the request. This transformation lets you pass requests as a method arguments, delay or queue a request's execution, and support undoable operations.

**Example:**
```javascript
class Calculator {
  constructor() { this.value = 0; }
  add(v) { this.value += v; }
  sub(v) { this.value -= v; }
}

class Command {
  constructor(subject) { this.subject = subject; this.commands = []; }
  execute(command, value) {
    this.subject[command](value);
    this.commands.push({ command, value });
  }
  undo() {
    const last = this.commands.pop();
    const reverse = last.command === "add" ? "sub" : "add";
    this.subject[reverse](last.value);
  }
}

const calc = new Calculator();
const cmd = new Command(calc);
cmd.execute("add", 100);
console.log(calc.value); // 100
cmd.undo();
console.log(calc.value); // 0
```

---

### 38. What is Dependency Injection (DI)?

**Definition:** **Dependency Injection** is a design pattern where an object receives its dependencies from an external source rather than creating them itself.

**Example:**
```javascript
// ❌ Hard-coded dependency (Tight coupling)
class UserNotification {
  send(msg) {
    const logger = new ConsoleLogger(); // Created inside
    logger.log(msg);
  }
}

// ✅ Dependency Injection (Loose coupling)
class UserNotificationDI {
  constructor(logger) {
    this.logger = logger; // Injected from outside
  }
  send(msg) {
    this.logger.log(msg);
  }
}

const notify = new UserNotificationDI(new SentryLogger());
```

**Benefits:** Easier testing (can inject mocks), flexibility, decoupled code.

---

### 39. What is the difference between MVC, MVP, and MVVM?

| Pattern | Description | Key Characteristic |
|---|---|---|
| **MVC** (Model-View-Controller) | Controller handles the user input and updates the Model. | View is passive and notified of changes. |
| **MVP** (Model-View-Presenter) | Presenter acts as a middleman between View and Model. | View and Model are completely decoupled. |
| **MVVM** (Model-View-ViewModel) | ViewModel exposes data from Model to View via Data Binding. | Automates the synchronization between View and Model (e.g., Vue, Angular, Knockout). |

---

### 40. What is Micro-Frontends architecture?

**Definition:** An architectural style where independently deliverable frontend applications are composed into a greater whole. It's the "Microservices for the frontend" approach.

**Key benefits:**
-   **Independent Deployments:** Teams can deploy their parts without affecting others.
-   **Technology Agnostic:** One team can use React, another Vue.
-   **Isolated Development:** Clear boundaries between different domain areas.

**Common implementation methods:**
-   **Iframe:** Simplest, but has SEO and performance drawbacks.
-   **Build-time integration:** Components shared via NPM.
-   **Runtime integration:** Using Module Federation (Webpack 5) or Single-SPA.

---

## 🚀 Section 5: Performance & Browser Internals

---

### 41. What are V8 Hidden Classes and Inline Caching?

**Definition:** JS is a dynamic language, but most JS engines like V8 use **Hidden Classes** (also called Shapes or Structures) to optimize property access, similar to how static languages like Java or C++ use fixed offsets.

**How it works:**
Every time you add a new property to an object, V8 creates a new hidden class and links the object to it.

```javascript
// Hidden class C0 (empty)
function Point(x, y) {
  this.x = x; // Hidden class C1 (C0 + 'x')
  this.y = y; // Hidden class C2 (C1 + 'y')
}

const p1 = new Point(1, 2); // Internal structure: Point { x, y } (C2)
const p2 = new Point(10, 20); // Internal structure: Point { x, y } (C2)
```

**Inline Caching (IC):**
When V8 sees code like `p.x`, it remembers the hidden class of `p` and the offset of `x`. Next time it sees `p.x`, if the hidden class matches, it jumps directly to the memory offset rather than doing a full lookup.

**Optimization Tip:**
**Always initialize properties in the same order.** If you change the order, V8 will create different hidden classes, breaking Inline Caching.

```javascript
// ❌ Anti-pattern: Random property order
const obj1 = {};
obj1.a = 1;
obj1.b = 2; // Class C_ab

const obj2 = {};
obj2.b = 2; 
obj2.a = 1; // Class C_ba (C_ab !== C_ba) -> IC won't work optimally!
```

---

### 42. What is the difference between JIT and Bytecode?

| Stage | Description |
|---|---|
| **Parsing** | Source code is turned into an Abstract Syntax Tree (AST). |
| **Interpreter (Ignition)** | AST is converted into **Bytecode**. This runs relatively fast but is not optimized. |
| **JIT Compiler (TurboFan)** | V8 monitors frequently run code ("Hot" code) and compiles that Bytecode into **Optimized Machine Code** (JIT). |
| **Deoptimization** | If the assumptions made for optimization become false (e.g., a function that always got integers suddenly gets a string), V8 throws away the machine code and goes back to Bytecode. |

---

### 43. What is a Memory Profile and how to read a Heap Snapshot?

**How to profile:**
Chrome DevTools -> Memory tab -> Take Heap Snapshot.

**Key Metrics:**
1.  **Shallow Size:** Memory used by the object itself (usually small for everything except arrays and strings).
2.  **Retained Size:** Memory that would be freed if this object (and everything only reachable through it) were deleted. This is the **most important metric** for finding memory leaks.
3.  **Distance:** Number of steps from the GC Root.

**Finding a leak:**
Take two snapshots before and after a suspicion (e.g., opening and closing a modal) and use the **Comparison** view to see which objects were allocated but not deleted.

---

### 44. What is the impact of excessive Closures on Performance?

**Memory Impact:**
As discussed in Q21, closures keep their outer scope alive. If you have millions of closures, you keep millions of objects from being garbage collected.

**Access Speed:**
Accessing a variable in a closure (outer scope) is slightly slower than accessing a local variable because the engine has to walk up the scope chain.

---

### 45. What is the Critical Rendering Path?

**Definition:** The sequence of steps the browser goes through to convert HTML, CSS, and JS into pixels on the screen.

**Steps:**
1.  **DOM Construction:** HTML -> DOM (Document Object Model).
2.  **CSSOM Construction:** CSS -> CSSOM (CSS Object Model).
3.  **Render Tree:** Combining DOM and CSSOM (ignores `display: none`).
4.  **Layout (Reflow):** Calculating the exact position and size of each node.
5.  **Paint:** Filling in pixels (text, colors, images, borders).
6.  **Composite:** Stacking layers together (useful for GPU-accelerated transforms).

**Optimizing for Performance:**
-   **Minimize Reflows:** Batch DOM updates, avoid layout-thrashing (reading then writing layout properties).
-   **Use `requestAnimationFrame`** for animations.
-   **CSS Containment:** Using `contain` property to prevent styles from leaking/affecting rest of the layout.

---

*End of JavaScript Advanced Reference*

---

### 46. What is Functional Programming (FP) in JavaScript?

**Definition:** **Functional Programming** is a declarative programming paradigm where programs are constructed by applying and composing functions. It treats computation as the evaluation of mathematical functions and avoids changing state and mutable data.

**Key Pillars of FP:**
1.  **First-Class Functions:** Functions are treated like any other variable (passed as arguments, returned from other functions).
2.  **Pure Functions:** Predictable output for the same input, with no side effects.
3.  **Immutability:** Data is never changed; instead, new data structures are created.
4.  **Composition:** Building complex logic by combining simpler functions.
5.  **Declarative vs Imperative:** Focus on *what* to do rather than *how* to do it.

---

### 47. What are Pure Functions and Side Effects?

**Pure Function:**
-   **Same Input → Same Output:** It doesn't depend on any external state (like a global variable or database).
-   **No Side Effects:** It doesn't modify anything outside of itself (doesn't change global variables, doesn't log to console, doesn't write to a DB).

**Example:**
```js
// ✅ Pure
const add = (a, b) => a + b;

// ❌ Impure (Side Effect)
let total = 0;
const addToTotal = (a) => {
  total += a; // Modifies external state
  return total;
};

// ❌ Impure (Non-deterministic)
const getRandom = (a) => a * Math.random(); // Different output for same input
```

---

### 48. What is Immutability and why is it important?

**Definition:** **Immutability** means that once a data structure is created, it cannot be changed. If you need to modify it, you create a new version with the changes.

**Why it matters in JS/React:**
1.  **Predictability:** You know your data won't "magically" change elsewhere in the app.
2.  **Performance (Change Detection):** In React, comparing two objects by reference (`oldObj === newObj`) is extremely fast. If you mutate an object, the reference stays the same, and React might not realize it needs to re-render.
3.  **Undo/Redo (Time Travel):** Since you keep old versions of state, implementing "undo" becomes trivial.

---

### 49. What is Function Composition (Pipe/Compose)?

**Definition:** Composition is the process of combining two or more functions to produce a new function. It's the "LEGO" approach to coding.

-   **Compose:** Executes from right-to-left. `f(g(x))`
-   **Pipe:** Executes from left-to-right. `g(f(x))`

**Example Example:**
```js
const multiplyBy2 = x => x * 2;
const add5 = x => x + 5;

// Manual composition
const result = add5(multiplyBy2(10)); // 25

// Using a pipe helper
const pipe = (...fns) => (val) => fns.reduce((acc, fn) => fn(acc), val);

const calculate = pipe(multiplyBy2, add5);
console.log(calculate(10)); // 25
```

---

### 50. What is Currying and Partial Application?

**Currying:** Transforming a function that takes multiple arguments into a sequence of functions that each take a single argument.
`f(a, b, c)` becomes `f(a)(b)(c)`.

**Partial Application:** Fixing a number of arguments to a function, producing another function of smaller arity.

**Example:**
```js
// Normal function
const multiply = (a, b) => a * b;

// Curried function
const curriedMultiply = (a) => (b) => a * b;

const double = curriedMultiply(2); // Partial application
console.log(double(5)); // 10
console.log(double(10)); // 20
```

---

### 51. Deep vs Shallow Copy (Internals)

-   **Shallow Copy:** Only copies the top-level properties. Nested objects or arrays are still **passed by reference**. Changes to nested data in the copy will affect the original.
    -   Methods: `Object.assign()`, Spread operator `[...]` / `{...}`.
-   **Deep Copy:** Recursively copies all levels of the object. The original and the copy share NO references.
    -   Methods: `JSON.parse(JSON.stringify(obj))` (fast but loses functions/Dates), `structuredClone()` (modern built-in), or libraries like `lodash.cloneDeep`.

---

### 52. What is "Proxy" in JavaScript?

**Definition:** The **Proxy** object allows you to create a "wrapper" for another object, which can intercept and redefine fundamental operations for that object (like getting/setting properties).

**Use Case: Reactivity (Vue 3 / MobX):**
```js
const user = { name: "Alice", age: 25 };

const proxyUser = new Proxy(user, {
  get(target, prop) {
    console.log(`Getting property: ${prop}`);
    return target[prop];
  },
  set(target, prop, value) {
    console.log(`Setting ${prop} to ${value}`);
    // You can add validation logic here
    if (prop === "age" && value < 0) throw new Error("Age cannot be negative");
    target[prop] = value;
    return true;
  }
});

proxyUser.age = 30; // Logs "Setting age to 30"
console.log(proxyUser.name); // Logs "Getting property: name", then "Alice"
```

---

### 53. What is the "Temporal Dead Zone" (TDZ)?

**Definition:** The **TDZ** is the period between the entering of a scope (where a variable is hoisted) and the actual declaration of that variable using `let` or `const`. Accessing the variable during this window throws a `ReferenceError`.

**Why does it exist?** To catch bugs early by preventing the use of variables before they are initialized (which was a big problem with `var` and its `undefined` behavior).

---

### 54. What is the difference between `for...in` and `for...of`?

-   **`for...in`**: Iterates over the **enumerable property keys** of an object (including inherited ones from the prototype chain). Best for objects.
-   **`for...of`**: Iterates over the **values** of an **iterable** object (Arrays, Strings, Maps, Sets). It uses the `[Symbol.iterator]` protocol.

---

### 55. What is "Memoization" as a general concept in JS?

**Definition:** Memoization is an optimization technique used to speed up computer programs by **storing the results of expensive function calls** and returning the cached result when the same inputs occur again.

**Manual Implementation:**
```js
function memoize(fn) {
  const cache = new Map();
  return function(...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) return cache.get(key);
    
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  }
}
```

---

*Expansion Complete*
