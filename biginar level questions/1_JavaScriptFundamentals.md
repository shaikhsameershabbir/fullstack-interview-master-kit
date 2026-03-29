# JavaScript Fundamentals – Complete Interview Reference

---

## 📦 Section 1: JavaScript Fundamentals

---

### 1. What is JavaScript?

JavaScript is a **lightweight, interpreted, single-threaded, dynamically-typed, multi-paradigm programming language**. Originally built to add interactivity to web pages in browsers, it now runs on servers (Node.js), mobile (React Native), and desktop (Electron).

**Key traits:**
- Runs inside the browser's **JS engine** (e.g., V8 in Chrome, SpiderMonkey in Firefox)
- Supports **OOP**, **functional**, and **procedural** programming styles
- Uses an **event loop** to handle asynchronous work (timers, HTTP, etc.)
- **Garbage collected** — memory is managed automatically

```js
console.log("Hello, World!"); // Output: Hello, World!
```

**ECMAScript (ES):** JavaScript follows the ECMAScript specification. Modern JS = ES6+ (2015 onwards).

---

### 2. What are the Primitive Data Types in JavaScript?

A **primitive** is an immutable value that is NOT an object (no methods of its own). There are **7 primitives** in JS:

| Type | Example | Notes |
|---|---|---|
| `string` | `"hello"` | Text data |
| `number` | `42`, `3.14`, `NaN`, `Infinity` | All numbers are 64-bit floats |
| `bigint` | `9007199254740991n` | Very large integers |
| `boolean` | `true`, `false` | Logical values |
| `undefined` | `let x;` | Variable declared but not assigned |
| `null` | `let x = null;` | Intentional empty value |
| `symbol` | `Symbol("id")` | Unique identity token |

Everything else (arrays, functions, dates, etc.) is an **object**.

```js
typeof "hello"     // "string"
typeof 42          // "number"
typeof true        // "boolean"
typeof undefined   // "undefined"
typeof null        // "object"  ← famous bug in JS!
typeof Symbol()    // "symbol"
typeof 42n         // "bigint"
```

---

### 3. What is the difference between `var`, `let`, and `const`?

| Feature | `var` | `let` | `const` |
|---|---|---|---|
| Scope | Function-scoped | Block-scoped | Block-scoped |
| Hoisting | Hoisted & initialized as `undefined` | Hoisted but NOT initialized (TDZ) | Hoisted but NOT initialized (TDZ) |
| Re-declaration | ✅ Allowed | ❌ Not allowed | ❌ Not allowed |
| Re-assignment | ✅ Allowed | ✅ Allowed | ❌ Not allowed |
| Global object property | Yes (`window.x`) | No | No |

```js
// var - function scoped
function test() {
  if (true) {
    var x = 10;
  }
  console.log(x); // 10 — leaks out of block!
}

// let - block scoped
function test2() {
  if (true) {
    let y = 20;
  }
  console.log(y); // ReferenceError!
}

// const - must be initialized, cannot be reassigned
const PI = 3.14;
PI = 3; // TypeError: Assignment to constant variable

// But objects/arrays can be mutated
const arr = [1, 2, 3];
arr.push(4); // OK — reference is const, not the contents
```

> **Best Practice:** Always prefer `const`. Use `let` only when you need to reassign. Avoid `var`.

---

### 4. What is Hoisting?

**Hoisting** is JavaScript's behavior of moving **declarations** (not initializations) to the top of their scope during the compilation phase — before code executes.

#### `var` hoisting:
```js
console.log(a); // undefined (not ReferenceError)
var a = 5;

// JS internally treats it as:
var a;           // declaration hoisted
console.log(a);  // undefined
a = 5;           // initialization stays
```

#### `let` / `const` — Temporal Dead Zone (TDZ):
```js
console.log(b); // ReferenceError: Cannot access 'b' before initialization
let b = 10;
// 'b' is hoisted but NOT initialized — lives in TDZ until declaration line
```

#### Function hoisting:
```js
greet(); // "Hello!" — works! Full function is hoisted

function greet() {
  console.log("Hello!");
}

// Function expression — NOT hoisted
sayHi(); // TypeError: sayHi is not a function
var sayHi = function() { console.log("Hi!"); };
```

---

### 5. What is a Closure?

A **closure** is a function that **remembers the variables from its outer (lexical) scope** even after the outer function has returned.

```js
function outer() {
  let count = 0; // this variable is "closed over"

  return function inner() {
    count++;
    console.log(count);
  };
}

const increment = outer();
increment(); // 1
increment(); // 2
increment(); // 3
// 'count' still lives in memory because 'inner' holds a reference to it
```

**Real-world uses of closures:**
- **Data privacy / encapsulation** (module pattern)
- **Factory functions**
- **Memoization / caching**
- **Event handlers** (callbacks remembering context)

```js
// Data privacy with closure
function createCounter() {
  let count = 0; // private!
  return {
    increment: () => ++count,
    decrement: () => --count,
    getCount: () => count,
  };
}
const counter = createCounter();
counter.increment(); // 1
counter.increment(); // 2
console.log(counter.getCount()); // 2
// count cannot be accessed directly from outside
```

---

### 6. What is the Event Loop?

JavaScript is **single-threaded** — it has one call stack and can do one thing at a time. The **event loop** is the mechanism that allows JS to perform **non-blocking async operations** despite this limitation.

**Components:**
1. **Call Stack** — where currently executing functions live (LIFO)
2. **Web APIs** — browser-provided APIs (setTimeout, fetch, DOM events) that run outside the JS engine
3. **Callback Queue (Macro-task queue)** — completed callbacks wait here (setTimeout, setInterval, DOM events)
4. **Microtask Queue** — higher-priority queue (Promise callbacks, `queueMicrotask`, `MutationObserver`)
5. **Event Loop** — continuously checks: if call stack is empty, push next item from microtask queue (first), then macro-task queue

```
Call Stack → empty?
   ↓ Yes
Check Microtask Queue → process ALL microtasks
   ↓ Empty
Check Macro-task Queue → process ONE callback
   ↓
Repeat
```

```js
console.log("1");

setTimeout(() => console.log("2"), 0); // macro-task

Promise.resolve().then(() => console.log("3")); // micro-task

console.log("4");

// Output: 1, 4, 3, 2
// Synchronous first → microtasks → macrotasks
```

---

### 7. What is the Call Stack?

The **call stack** is a **LIFO (Last In, First Out)** data structure that keeps track of function calls currently being executed.

- When a function is invoked → pushed onto the stack
- When it returns → popped off the stack
- If the stack overflows (too many recursive calls) → `Maximum call stack size exceeded`

```js
function a() {
  b();
}
function b() {
  c();
}
function c() {
  console.log("inside c");
}
a();

// Stack at deepest point:
// [ c, b, a, main ]
// c returns → [ b, a, main ]
// b returns → [ a, main ]
// a returns → [ main ]
```

**Stack overflow:**
```js
function recurse() {
  return recurse(); // no base case!
}
recurse(); // Uncaught RangeError: Maximum call stack size exceeded
```

---

### 8. Difference between `==` and `===`?

| Operator | Name | Type Coercion | Check |
|---|---|---|---|
| `==` | Loose equality | ✅ Yes (coerces types) | Value only |
| `===` | Strict equality | ❌ No | Value AND type |

```js
0 == false       // true  (false coerced to 0)
0 === false      // false (different types)

"" == false      // true  (both coerce to 0)
"" === false     // false

null == undefined  // true  (special rule)
null === undefined // false

1 == "1"         // true  ("1" coerced to 1)
1 === "1"        // false
```

> **Best Practice:** Always use `===` unless you explicitly need type coercion.

---

### 9. What is `undefined` vs `null`?

| | `undefined` | `null` |
|---|---|---|
| Meaning | Variable declared but no value assigned | Intentional absence of value |
| Set by | JavaScript automatically | Developer explicitly |
| Type | `"undefined"` | `"object"` (JS bug) |
| Use case | Uninitialized state | Empty/no object intentionally |

```js
let a;
console.log(a);           // undefined
console.log(typeof a);    // "undefined"

let b = null;
console.log(b);           // null
console.log(typeof b);    // "object"

// Equality
null == undefined   // true  (loose)
null === undefined  // false (strict)
```

---

### 10. What is `NaN`?

`NaN` stands for **"Not a Number"**. It is a value of type `number` that represents an illegal or undefined mathematical result.

```js
typeof NaN  // "number" (counterintuitive but correct)

// When does NaN appear?
0 / 0            // NaN
parseInt("abc")  // NaN
Math.sqrt(-1)    // NaN
undefined + 1    // NaN

// NaN is the ONLY value that is NOT equal to itself
NaN === NaN  // false
NaN == NaN   // false

// Correct way to check for NaN:
Number.isNaN(NaN)       // true ✅
Number.isNaN("hello")   // false (does not coerce)
isNaN("hello")          // true  (coerces first — less reliable)
```

---
## 🔧 Section 2: JavaScript Functions

---

### 11. What is a Function Declaration?

A **function declaration** (also called a function statement) defines a named function using the `function` keyword. It is **fully hoisted**.

```js
// Can be called BEFORE it is defined (hoisting)
greet("Alice"); // "Hello, Alice!"

function greet(name) {
  return `Hello, ${name}!`;
}
```

- Has its own `this` context
- Can be recursive by name
- Adds the function to the current scope

---

### 12. What is a Function Expression?

A **function expression** assigns a function (named or anonymous) to a variable. It is **NOT fully hoisted** — only the variable is hoisted, not the function body.

```js
// Anonymous function expression
const add = function(a, b) {
  return a + b;
};

// Named function expression (name only available inside itself)
const factorial = function fact(n) {
  return n <= 1 ? 1 : n * fact(n - 1);
};

console.log(add(2, 3));       // 5
console.log(factorial(5));    // 120
```

**Key difference from declaration:**
```js
sayHello(); // TypeError: sayHello is not a function
var sayHello = function() { console.log("Hi!"); };
```

---

### 13. What is an Arrow Function?

Arrow functions are a **concise ES6 syntax** for writing functions. Key difference: they **do NOT have their own `this`** — they inherit `this` from the surrounding lexical scope.

```js
// Traditional
const square = function(x) { return x * x; };

// Arrow function
const square = (x) => x * x;

// Multi-line arrow function
const add = (a, b) => {
  const result = a + b;
  return result;
};

// No params
const greet = () => "Hello!";

// Single param — parens optional
const double = x => x * 2;
```

**`this` behavior:**
```js
const obj = {
  name: "Alice",
  greetTraditional: function() {
    console.log(this.name); // "Alice" ✅
  },
  greetArrow: () => {
    console.log(this.name); // undefined ❌ (inherits from outer scope = window/undefined)
  }
};
```

**Cannot be used as:**
- Constructors (`new arrowFn()` throws TypeError)
- Generator functions
- Methods when `this` binding is needed

---

### 14. What are Higher-Order Functions?

A **higher-order function (HOF)** is a function that either:
1. **Takes a function as an argument**, or
2. **Returns a function**

```js
// Takes a function as argument
function applyTwice(fn, value) {
  return fn(fn(value));
}
applyTwice(x => x + 3, 10); // 16

// Returns a function
function multiplier(factor) {
  return (number) => number * factor;
}
const double = multiplier(2);
const triple = multiplier(3);
double(5); // 10
triple(5); // 15
```

**Built-in HOFs:**
```js
const nums = [1, 2, 3, 4, 5];

nums.map(x => x * 2);        // [2, 4, 6, 8, 10]
nums.filter(x => x % 2 === 0); // [2, 4]
nums.reduce((acc, x) => acc + x, 0); // 15
nums.forEach(x => console.log(x));
nums.find(x => x > 3);       // 4
nums.some(x => x > 4);       // true
nums.every(x => x > 0);      // true
```

---

### 15. What is a Callback Function?

A **callback** is a function **passed as an argument** to another function, to be **called later** — either synchronously or asynchronously.

```js
// Synchronous callback
function greet(name, callback) {
  console.log("Hello, " + name);
  callback();
}

greet("Alice", () => console.log("Callback executed!"));
// Hello, Alice
// Callback executed!

// Asynchronous callback
setTimeout(() => {
  console.log("Runs after 1 second");
}, 1000);

// Node.js style (error-first callbacks)
fs.readFile("file.txt", "utf8", (err, data) => {
  if (err) return console.error(err);
  console.log(data);
});
```

**Callback Hell (Pyramid of Doom):**
```js
getUser(id, (user) => {
  getOrders(user.id, (orders) => {
    getItems(orders[0].id, (items) => {
      // deeply nested — hard to read/maintain
    });
  });
});
// Solved by: Promises or async/await
```

---

### 16. What is an IIFE?

**IIFE** = Immediately Invoked Function Expression. A function that is **defined and called immediately**.

```js
// Basic IIFE
(function() {
  console.log("I run immediately!");
})();

// Arrow IIFE
(() => {
  console.log("Arrow IIFE!");
})();

// IIFE with arguments
(function(name) {
  console.log(`Hello, ${name}!`);
})("Alice");
```

**Why use IIFEs?**
- **Avoid polluting global scope** — variables inside are private
- **Module pattern** before ES modules existed
- **One-time initialization** code

```js
const counter = (function() {
  let count = 0; // private
  return {
    increment: () => ++count,
    reset: () => (count = 0),
    get: () => count,
  };
})();

counter.increment(); // 1
counter.increment(); // 2
counter.get();       // 2
```

---

### 17. What is Currying?

**Currying** is transforming a function that takes **multiple arguments** into a sequence of functions that each take **one argument**.

```js
// Normal function
function add(a, b, c) {
  return a + b + c;
}
add(1, 2, 3); // 6

// Curried version
function curriedAdd(a) {
  return function(b) {
    return function(c) {
      return a + b + c;
    };
  };
}

curriedAdd(1)(2)(3); // 6

// Arrow function shorthand
const curried = a => b => c => a + b + c;
curried(1)(2)(3); // 6
```

**Practical use — partial application:**
```js
const multiply = a => b => a * b;

const double = multiply(2);  // partially applied
const triple = multiply(3);

double(5); // 10
triple(5); // 15
```

---

### 18. What is Function Binding?

**`bind()`** creates a **new function** with a permanently bound `this` and optionally pre-filled arguments.

```js
const person = { name: "Alice" };

function greet(greeting, punctuation) {
  console.log(`${greeting}, ${this.name}${punctuation}`);
}

const boundGreet = greet.bind(person, "Hello");
boundGreet("!"); // "Hello, Alice!"

// call() — invokes immediately, args as comma-separated
greet.call(person, "Hi", "!"); // "Hi, Alice!"

// apply() — invokes immediately, args as array
greet.apply(person, ["Hey", "."]); // "Hey, Alice."
```

| Method | Invokes immediately | Args format | Returns |
|---|---|---|---|
| `bind()` | ❌ | comma-separated | new function |
| `call()` | ✅ | comma-separated | result |
| `apply()` | ✅ | array | result |

---

### 19. What are Default Parameters?

ES6 allows function parameters to have **default values** if no argument (or `undefined`) is passed.

```js
function greet(name = "World", greeting = "Hello") {
  return `${greeting}, ${name}!`;
}

greet();              // "Hello, World!"
greet("Alice");       // "Hello, Alice!"
greet("Bob", "Hi");   // "Hi, Bob!"
greet(undefined, "Hey"); // "Hey, World!" (undefined triggers default)
greet(null, "Hey");      // "Hey, null!"  (null does NOT trigger default)
```

Defaults can be expressions or other function calls:
```js
function createId(prefix = "user", id = Date.now()) {
  return `${prefix}_${id}`;
}
```

---

### 20. What is the Rest and Spread Operator?

Both use `...` syntax but serve opposite purposes.

#### Rest (`...`) — collects multiple values into an array:
```js
function sum(...numbers) {
  return numbers.reduce((acc, n) => acc + n, 0);
}
sum(1, 2, 3, 4); // 10

// Must be last parameter
function log(first, ...rest) {
  console.log(first); // first arg
  console.log(rest);  // array of remaining
}
log(1, 2, 3, 4); // 1, [2, 3, 4]
```

#### Spread (`...`) — expands an iterable into individual elements:
```js
// Arrays
const a = [1, 2, 3];
const b = [4, 5, 6];
const merged = [...a, ...b]; // [1, 2, 3, 4, 5, 6]

// Copy array
const copy = [...a];

// Function call
Math.max(...a); // 3

// Objects
const obj1 = { x: 1, y: 2 };
const obj2 = { y: 10, z: 3 };
const merged2 = { ...obj1, ...obj2 }; // { x: 1, y: 10, z: 3 }
```

---

## 🧩 Section 3: JavaScript Objects

---

### 21. What is an Object in JavaScript?

An **object** is a collection of **key-value pairs** (properties). Values can be any data type, including functions (called methods). Objects are **reference types** in JS.

```js
// Object literal
const person = {
  name: "Alice",
  age: 25,
  isAdmin: false,
  address: { city: "Mumbai", zip: "400001" }, // nested object
  greet() {                                    // method (shorthand)
    return `Hi, I'm ${this.name}`;
  }
};

// Accessing properties
person.name;           // dot notation → "Alice"
person["age"];         // bracket notation → 25
person.greet();        // "Hi, I'm Alice"

// Adding / modifying properties
person.email = "alice@mail.com";
person.age = 26;

// Deleting a property
delete person.isAdmin;

// Checking property existence
"name" in person;       // true
person.hasOwnProperty("name"); // true
```

---

### 22. What is Object Destructuring?

**Destructuring** allows you to **extract properties** from an object into named variables in a single statement.

```js
const user = { name: "Alice", age: 25, city: "Mumbai" };

// Basic destructuring
const { name, age } = user;
console.log(name); // "Alice"
console.log(age);  // 25

// Rename while destructuring
const { name: userName, age: userAge } = user;
console.log(userName); // "Alice"

// Default values
const { role = "user" } = user;
console.log(role); // "user" (not in object, uses default)

// Nested destructuring
const { address: { city } = {} } = user; // safe with fallback

// In function parameters
function display({ name, age }) {
  console.log(`${name} is ${age}`);
}
display(user); // "Alice is 25"

// Array destructuring
const [first, second, , fourth] = [10, 20, 30, 40];
console.log(first, fourth); // 10, 40

// Swap variables
let a = 1, b = 2;
[a, b] = [b, a];
console.log(a, b); // 2, 1
```

---

### 23. What is Prototype?

Every JavaScript object has a hidden internal property called `[[Prototype]]` (accessible via `__proto__` or `Object.getPrototypeOf()`). It points to **another object** from which it inherits properties and methods.

```js
const animal = {
  eat() { console.log("eating"); }
};

const dog = Object.create(animal); // dog.[[Prototype]] = animal
dog.bark = function() { console.log("woof!"); };

dog.bark(); // "woof!"  — own method
dog.eat();  // "eating" — inherited from animal via prototype chain

// Checking prototype
Object.getPrototypeOf(dog) === animal; // true

// Every object ultimately inherits from Object.prototype
// Object.prototype.[[Prototype]] = null (end of chain)
```

**Prototype chain lookup:**
1. Look on the object itself
2. Look on its `[[Prototype]]`
3. Continue up the chain until `null`

---

### 24. What is Prototypal Inheritance?

**Prototypal inheritance** is JavaScript's mechanism where objects inherit directly from other objects (not from classes, despite the ES6 `class` syntax being sugar over this).

```js
// Constructor function pattern
function Animal(name) {
  this.name = name;
}
Animal.prototype.speak = function() {
  console.log(`${this.name} makes a sound`);
};

function Dog(name, breed) {
  Animal.call(this, name); // call parent constructor
  this.breed = breed;
}

// Set up inheritance chain
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;

Dog.prototype.bark = function() {
  console.log("Woof!");
};

const d = new Dog("Rex", "Labrador");
d.speak(); // "Rex makes a sound" ← inherited
d.bark();  // "Woof!"

// ES6 class syntax (same under the hood)
class Animal {
  constructor(name) { this.name = name; }
  speak() { console.log(`${this.name} makes a sound`); }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name); // calls Animal constructor
    this.breed = breed;
  }
  bark() { console.log("Woof!"); }
}
```

---

### 25. What is the `this` Keyword?

`this` refers to the **object that is currently executing the function**. Its value depends on **how** the function is called, not where it is defined.

```js
// 1. Global context
console.log(this); // window (browser) / global (Node)

// 2. Object method
const obj = {
  name: "Alice",
  greet() { console.log(this.name); } // this = obj
};
obj.greet(); // "Alice"

// 3. Regular function (non-strict mode)
function show() { console.log(this); } // this = window
show();

// 4. Arrow function — inherits this from enclosing scope
const obj2 = {
  name: "Bob",
  greet: () => console.log(this.name) // this = outer scope (not obj2!)
};

// 5. Constructor (new keyword)
function Person(name) {
  this.name = name; // this = new object created
}
const p = new Person("Alice");
p.name; // "Alice"

// 6. Explicit binding
function greet() { console.log(this.name); }
greet.call({ name: "Charlie" }); // "Charlie"
greet.apply({ name: "Diana" });  // "Diana"
const bound = greet.bind({ name: "Eve" });
bound(); // "Eve"
```

---

### 26. What are Getters and Setters?

Getters and setters allow you to define **computed properties** and intercept property access/mutation.

```js
const circle = {
  _radius: 5, // convention: _ means "private"

  get radius() {
    return this._radius;
  },
  set radius(value) {
    if (value < 0) throw new Error("Radius must be positive");
    this._radius = value;
  },
  get area() {
    return Math.PI * this._radius ** 2; // computed, no setter needed
  }
};

circle.radius;       // 5 (calls getter)
circle.radius = 10;  // calls setter
circle.area;         // 314.159... (computed)
circle.radius = -1;  // throws Error!

// In class syntax
class Temperature {
  constructor(celsius) { this._celsius = celsius; }

  get fahrenheit() { return this._celsius * 9/5 + 32; }
  set fahrenheit(f) { this._celsius = (f - 32) * 5/9; }
}

const t = new Temperature(0);
t.fahrenheit;       // 32
t.fahrenheit = 212; // sets celsius to 100
```

---

### 27. What is `Object.freeze()`?

`Object.freeze()` makes an object **immutable** — properties cannot be added, removed, or modified.

```js
const config = Object.freeze({
  host: "localhost",
  port: 3000,
  db: { name: "mydb" } // nested objects are NOT deeply frozen!
});

config.port = 8080;     // silently ignored (or TypeError in strict mode)
config.newProp = "hi";  // silently ignored
delete config.host;     // silently ignored

console.log(config.port); // 3000 — unchanged

// Shallow freeze only!
config.db.name = "changed"; // ✅ This WORKS — nested object not frozen

// Deep freeze (custom utility)
function deepFreeze(obj) {
  Object.keys(obj).forEach(key => {
    if (typeof obj[key] === "object") deepFreeze(obj[key]);
  });
  return Object.freeze(obj);
}
```

---

### 28. What is `Object.assign()`?

`Object.assign(target, ...sources)` **copies enumerable own properties** from one or more source objects into a target object. Returns the target. It performs a **shallow copy**.

```js
const target = { a: 1, b: 2 };
const source = { b: 4, c: 5 };

const result = Object.assign(target, source);
console.log(result); // { a: 1, b: 4, c: 5 } — target is mutated!
console.log(target === result); // true

// Clone an object (shallow)
const clone = Object.assign({}, source);

// Merge multiple sources
const merged = Object.assign({}, obj1, obj2, obj3);

// Preferred modern alternative: spread operator
const merged2 = { ...obj1, ...obj2 }; // same but cleaner
```

---

### 29. What is Shallow vs Deep Copy?

**Shallow copy:** Creates a new object, but **nested objects are still referenced** (not cloned).

**Deep copy:** Creates a completely independent clone — **all nested objects** are also copied.

```js
const original = { a: 1, nested: { b: 2 } };

// SHALLOW COPY METHODS:
const shallow1 = Object.assign({}, original);
const shallow2 = { ...original };

shallow1.a = 99;           // does NOT affect original (primitive)
shallow1.nested.b = 99;    // DOES affect original! (same reference)

console.log(original.a);        // 1 (safe)
console.log(original.nested.b); // 99 (mutated!)

// DEEP COPY METHODS:
// 1. JSON (simple, but lossy — no functions, undefined, Date, etc.)
const deep1 = JSON.parse(JSON.stringify(original));

// 2. structuredClone (modern, recommended)
const deep2 = structuredClone(original);

// 3. Recursion (manual)
function deepClone(obj) {
  if (obj === null || typeof obj !== "object") return obj;
  if (Array.isArray(obj)) return obj.map(deepClone);
  return Object.fromEntries(
    Object.entries(obj).map(([k, v]) => [k, deepClone(v)])
  );
}
```

---

### 30. What are Symbols?

`Symbol` is a **unique, immutable primitive** used primarily as object property keys to avoid name collisions.

```js
const id1 = Symbol("id");
const id2 = Symbol("id");

id1 === id2; // false — each Symbol is unique!

const user = {
  name: "Alice",
  [id1]: 123,    // symbol as object key (computed property)
};

user[id1];                // 123
user.name;                // "Alice"
Object.keys(user);        // ["name"] — symbols are hidden!
Object.getOwnPropertySymbols(user); // [Symbol(id)]

// Well-known Symbols
// Symbol.iterator — makes objects iterable
const myRange = {
  from: 1,
  to: 5,
  [Symbol.iterator]() {
    let current = this.from;
    const last = this.to;
    return {
      next() {
        return current <= last
          ? { value: current++, done: false }
          : { done: true };
      }
    };
  }
};

for (const n of myRange) console.log(n); // 1, 2, 3, 4, 5
```

---

## ⚡ Section 4: Asynchronous JavaScript

---

### 31. What is Asynchronous Programming?

**Asynchronous programming** allows code to start a long-running task (like a network request) and continue executing other code **without waiting** for it to finish. The result is handled via callbacks, Promises, or async/await.

```js
// SYNCHRONOUS - blocks execution
const data = readFileSync("file.txt"); // waits here
console.log(data);
console.log("This runs AFTER file is read");

// ASYNCHRONOUS - non-blocking
readFile("file.txt", (err, data) => {
  console.log(data); // runs when file is ready
});
console.log("This runs IMMEDIATELY, before file is read");
```

**Common async operations:**
- Network requests (`fetch`, `XMLHttpRequest`)
- File I/O (Node.js `fs`)
- Timers (`setTimeout`, `setInterval`)
- Database queries
- User events (click, input)

---

### 32. What is a Promise?

A **Promise** is an object representing the **eventual completion or failure** of an async operation. It provides a cleaner alternative to callback hell.

```js
// Creating a Promise
const fetchData = new Promise((resolve, reject) => {
  const success = true;

  setTimeout(() => {
    if (success) {
      resolve({ id: 1, name: "Alice" }); // fulfilled
    } else {
      reject(new Error("Network error"));  // rejected
    }
  }, 1000);
});

// Consuming a Promise
fetchData
  .then(data => {
    console.log("Got:", data); // { id: 1, name: "Alice" }
    return data.name;          // chain: next .then receives this
  })
  .then(name => console.log("Name:", name)) // "Alice"
  .catch(err => console.error("Error:", err.message))
  .finally(() => console.log("Done!")); // always runs
```

---

### 33. What are Promise States?

A Promise can be in exactly **one of three states**:

| State | Description | Transition |
|---|---|---|
| `pending` | Initial state — operation ongoing | → `fulfilled` or `rejected` |
| `fulfilled` | Operation succeeded | calls `.then()` handler |
| `rejected` | Operation failed | calls `.catch()` handler |

```js
const p = new Promise((resolve, reject) => {
  // currently "pending"
  setTimeout(() => resolve("done"), 1000);
});

// After 1 second: state = "fulfilled", value = "done"

// A settled promise (fulfilled or rejected) is IMMUTABLE
// Once resolved/rejected, it cannot change state
```

**Promise chaining:**
```js
fetch("/api/user")
  .then(res => res.json())       // transforms response
  .then(user => fetch(`/api/orders/${user.id}`))
  .then(res => res.json())
  .then(orders => console.log(orders))
  .catch(err => console.error(err)); // catches ANY error in chain
```

---

### 34. What is `async`/`await`?

`async`/`await` is **syntactic sugar over Promises** that lets you write async code that looks synchronous. An `async` function always returns a Promise.

```js
// Promise-based
function fetchUser(id) {
  return fetch(`/api/users/${id}`)
    .then(res => res.json())
    .then(user => user);
}

// async/await equivalent (much cleaner!)
async function fetchUser(id) {
  const res = await fetch(`/api/users/${id}`); // pauses here
  const user = await res.json();               // pauses here
  return user; // auto-wrapped in Promise.resolve(user)
}

// Error handling with try/catch
async function getData() {
  try {
    const user = await fetchUser(1);
    const orders = await fetchOrders(user.id);
    return orders;
  } catch (err) {
    console.error("Failed:", err.message);
  } finally {
    console.log("Always runs");
  }
}

// await can only be used inside async functions (or top-level in modules)
```

**Parallel execution with async/await:**
```js
// Sequential (slow — each waits for previous)
const a = await taskA();
const b = await taskB();

// Parallel (fast — run together)
const [a, b] = await Promise.all([taskA(), taskB()]);
```

---

### 35. What is `Promise.all()`?

`Promise.all()` takes an **array of Promises** and returns a single Promise that:
- **Resolves** when ALL Promises resolve (with an array of results)
- **Rejects** immediately if ANY Promise rejects (fail-fast)

```js
const p1 = fetch("/api/users").then(r => r.json());
const p2 = fetch("/api/products").then(r => r.json());
const p3 = fetch("/api/orders").then(r => r.json());

// Run all in parallel
const [users, products, orders] = await Promise.all([p1, p2, p3]);

// If ANY fails, the whole thing rejects
// Use for: independent operations that all must succeed
```

---

### 36. What is `Promise.race()`?

`Promise.race()` returns a Promise that **resolves or rejects** as soon as the **first** Promise in the array settles.

```js
const timeout = new Promise((_, reject) =>
  setTimeout(() => reject(new Error("Timeout!")), 3000)
);
const fetchData = fetch("/api/data").then(r => r.json());

// Whichever settles first wins
const result = await Promise.race([fetchData, timeout]);

// Other Promise combinators:
// Promise.allSettled() — waits for ALL, never rejects
// Promise.any()        — resolves on FIRST fulfillment, rejects if ALL fail
```

| Method | Resolves when | Rejects when |
|---|---|---|
| `Promise.all()` | ALL resolve | ANY rejects |
| `Promise.race()` | FIRST settles | FIRST rejects |
| `Promise.allSettled()` | ALL settle | Never |
| `Promise.any()` | FIRST resolves | ALL reject |

---

### 37. What are Microtasks?

**Microtasks** are tasks in the **microtask queue** — a high-priority queue processed **before** the next macro-task and **after** the current call stack is empty.

Sources of microtasks:
- `Promise.then()` / `.catch()` / `.finally()` callbacks
- `queueMicrotask()`
- `MutationObserver` callbacks

```js
console.log("1 - sync");

Promise.resolve().then(() => console.log("2 - microtask"));

setTimeout(() => console.log("3 - macrotask"), 0);

console.log("4 - sync");

// Output:
// 1 - sync
// 4 - sync
// 2 - microtask   ← microtask queue processed before macrotask
// 3 - macrotask
```

**The microtask queue is completely drained before the next macrotask begins**, even if new microtasks are added during processing.

---

### 38. What are Macrotasks?

**Macrotasks** (also called **tasks**) are in the regular task queue. After each macrotask, the event loop checks and drains the microtask queue before picking the next macrotask.

Sources of macrotasks:
- `setTimeout()`
- `setInterval()`
- `setImmediate()` (Node.js)
- I/O callbacks
- UI rendering events
- `MessageChannel`

```js
// Full event loop order demonstration:
console.log("Script start");       // sync

setTimeout(() => console.log("setTimeout"), 0);  // macrotask

Promise.resolve()
  .then(() => console.log("Promise 1"))           // microtask
  .then(() => console.log("Promise 2"));          // microtask

console.log("Script end");         // sync

// Output:
// Script start
// Script end
// Promise 1
// Promise 2
// setTimeout
```

---

### 39. What is `setTimeout`?

`setTimeout(callback, delay, ...args)` schedules a function to run **once** after at least `delay` milliseconds.

```js
// Basic usage
const timerId = setTimeout(() => {
  console.log("Runs after ~1 second");
}, 1000);

// With arguments
setTimeout((name, age) => {
  console.log(`${name} is ${age}`);
}, 500, "Alice", 25);

// Cancel before it fires
clearTimeout(timerId);

// delay = 0 does NOT mean immediate — it means "next macrotask"
setTimeout(() => console.log("async"), 0);
console.log("sync");
// Output: "sync" first, then "async"
```

> **Note:** The actual delay may be longer than specified if the call stack is busy. `delay` is a minimum, not a guarantee.

---

### 40. What is `setInterval`?

`setInterval(callback, delay, ...args)` **repeatedly** calls a function every `delay` milliseconds until stopped.

```js
let count = 0;
const intervalId = setInterval(() => {
  count++;
  console.log(`Tick: ${count}`);
  if (count >= 5) clearInterval(intervalId); // stop after 5
}, 1000);

// Clearing the interval
clearInterval(intervalId);

// Problem: setInterval doesn't wait for callback to finish
// Use recursive setTimeout for precise control:
function preciseTimer() {
  doSomething(); // if this takes longer than 1s, they won't overlap
  setTimeout(preciseTimer, 1000);
}
setTimeout(preciseTimer, 1000);
```

---

## 🌟 Section 5: Additional Important Questions

---

### 41. What is the `typeof` Operator?

`typeof` returns a **string** indicating the type of the operand.

```js
typeof "hello"      // "string"
typeof 42           // "number"
typeof 42n          // "bigint"
typeof true         // "boolean"
typeof undefined    // "undefined"
typeof Symbol()     // "symbol"
typeof function(){} // "function"
typeof {}           // "object"
typeof []           // "object"  ← arrays are objects
typeof null         // "object"  ← infamous JS bug!

// Better array check:
Array.isArray([]);          // true
value instanceof Array;     // true

// Better null check:
value === null;             // true
```

---

### 42. What is Scope in JavaScript?

**Scope** determines where variables and functions are accessible.

```js
// 1. Global Scope — accessible everywhere
var globalVar = "I'm global";

// 2. Function Scope — only inside the function
function myFn() {
  var local = "only here";
}
console.log(local); // ReferenceError

// 3. Block Scope — only inside { } (let, const)
{
  let blockVar = "block";
  const also = "block";
}
console.log(blockVar); // ReferenceError

// 4. Lexical (Static) Scope — functions use the scope where they were DEFINED
function outer() {
  const x = 10;
  function inner() {
    console.log(x); // 10 — has access to outer's scope
  }
  inner();
}
```

---

### 43. What is a Pure Function?

A **pure function** always returns the **same output for the same inputs** and has **no side effects**.

```js
// Pure function ✅
function add(a, b) { return a + b; }

// Impure — reads external state ❌
let tax = 0.1;
function total(price) { return price + price * tax; }

// Impure — mutates argument ❌
function addItem(arr, item) { arr.push(item); return arr; }

// Pure version — no mutation ✅
function addItem(arr, item) { return [...arr, item]; }
```

Benefits: predictable, testable, cacheable (memoizable), parallelizable.

---

### 44. What is Memoization?

**Memoization** is caching the results of a function based on its arguments to avoid redundant computation.

```js
function memoize(fn) {
  const cache = new Map();
  return function(...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) {
      console.log("From cache");
      return cache.get(key);
    }
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

function expensiveCalc(n) {
  // imagine heavy computation
  return n * n;
}

const memo = memoize(expensiveCalc);
memo(5); // computes 25
memo(5); // "From cache" → 25
memo(6); // computes 36
```

---

### 45. What is the Difference Between `map`, `filter`, and `reduce`?

```js
const numbers = [1, 2, 3, 4, 5];

// map — transforms each element, returns new array of same length
const doubled = numbers.map(n => n * 2); // [2, 4, 6, 8, 10]

// filter — keeps elements that pass a test, returns new array
const evens = numbers.filter(n => n % 2 === 0); // [2, 4]

// reduce — accumulates values into a single result
const sum = numbers.reduce((acc, n) => acc + n, 0); // 15

// Chaining
const result = numbers
  .filter(n => n % 2 !== 0)  // [1, 3, 5]
  .map(n => n * 10)           // [10, 30, 50]
  .reduce((a, b) => a + b, 0); // 90
```

---

### 46. What is `localStorage` vs `sessionStorage` vs Cookies?

| Feature | `localStorage` | `sessionStorage` | Cookies |
|---|---|---|---|
| Capacity | ~5MB | ~5MB | ~4KB |
| Expiry | Never (manual) | Tab close | Configurable |
| Server access | ❌ | ❌ | ✅ (sent with requests) |
| Scope | Origin-wide | Tab-specific | Configurable |
| API | JS only | JS only | JS + HTTP headers |

```js
// localStorage
localStorage.setItem("token", "abc123");
localStorage.getItem("token");    // "abc123"
localStorage.removeItem("token");
localStorage.clear();

// sessionStorage (same API)
sessionStorage.setItem("tab", "home");

// Cookies
document.cookie = "user=Alice; expires=Fri, 31 Dec 2025 23:59:59 GMT; path=/";
```

---

### 47. What is the Difference Between `==` Coercion Rules?

```js
// Abstract Equality (==) coercion rules:
// Rule 1: null == undefined → true (only these two)
null == undefined  // true
null == false      // false
null == 0          // false

// Rule 2: number vs string → string converted to number
"5" == 5           // true  ("5" → 5)

// Rule 3: boolean → converted to number
true == 1          // true  (true → 1)
false == 0         // true  (false → 0)
true == "1"        // true  (true → 1, "1" → 1)

// Rule 4: object vs primitive → object.valueOf() or .toString()
[] == false        // true  ([] → "" → 0, false → 0)
[] == 0            // true
{} == "[object Object]"  // false (but tricky)
```

---

### 48. What is Event Delegation?

**Event delegation** is attaching a **single event listener to a parent element** to handle events for multiple children, leveraging event bubbling.

```js
// Without delegation — listener per child (expensive)
document.querySelectorAll("li").forEach(li => {
  li.addEventListener("click", handleClick);
});

// With delegation — one listener on parent
document.getElementById("list").addEventListener("click", (e) => {
  if (e.target.tagName === "LI") {
    console.log("Clicked:", e.target.textContent);
  }
});

// Benefits:
// - Works for dynamically added elements
// - Better memory performance
// - Fewer event listeners
```

---

### 49. What is `for...of` vs `for...in`?

```js
const arr = [10, 20, 30];
const obj = { a: 1, b: 2, c: 3 };

// for...of — iterates VALUES of iterables (arrays, strings, Sets, Maps)
for (const val of arr) {
  console.log(val); // 10, 20, 30
}
for (const char of "hello") {
  console.log(char); // h, e, l, l, o
}

// for...in — iterates KEYS of an object (enumerable properties)
for (const key in obj) {
  console.log(key, obj[key]); // a 1, b 2, c 3
}

// Gotcha: for...in on arrays iterates indices (as strings)
for (const i in arr) {
  console.log(i); // "0", "1", "2" — keys, not values!
}
// Use for...of for arrays, for...in for plain objects
```

---

### 50. What are `Map` and `Set` in JavaScript?

#### `Map` — key-value pairs where keys can be any type:
```js
const map = new Map();
map.set("name", "Alice");
map.set(42, "answer");
map.set({ id: 1 }, "object key"); // objects as keys!

map.get("name");   // "Alice"
map.has(42);       // true
map.size;          // 3
map.delete("name");

// Iteration
for (const [key, value] of map) {
  console.log(key, "→", value);
}
```

#### `Set` — collection of unique values:
```js
const set = new Set([1, 2, 2, 3, 3, 3]);
console.log([...set]); // [1, 2, 3] — duplicates removed!

set.add(4);
set.has(2);   // true
set.delete(1);
set.size;     // 3

// Common use: remove duplicates from array
const unique = [...new Set([1, 2, 2, 3, 4, 4])]; // [1, 2, 3, 4]
```

---

### 51. What is Optional Chaining (`?.`)?

**Optional chaining** (`?.`) short-circuits and returns `undefined` instead of throwing an error when accessing a property on `null` or `undefined`.

```js
const user = {
  name: "Alice",
  address: {
    city: "Mumbai"
  }
};

// Without optional chaining — can crash
user.address.city;   // "Mumbai"
user.phone.number;   // TypeError: Cannot read property 'number' of undefined

// With optional chaining — safe
user?.phone?.number;         // undefined (no error)
user?.address?.city;         // "Mumbai"
user?.greet?.();             // undefined (safe method call)
user?.items?.[0];            // undefined (safe array access)

// With nullish coalescing
const city = user?.address?.city ?? "Unknown";
```

---

### 52. What is the Nullish Coalescing Operator (`??`)?

`??` returns the **right-hand side** value only when the left side is `null` or `undefined` (NOT for other falsy values like `0`, `""`, `false`).

```js
// || uses any falsy value as trigger
0 || "default"        // "default" (wrong — 0 is valid!)
"" || "default"       // "default" (wrong — empty string valid!)
false || "default"    // "default"

// ?? only triggers for null/undefined
0 ?? "default"        // 0      ✅ (zero is valid)
"" ?? "default"       // ""     ✅ (empty string is valid)
false ?? "default"    // false  ✅
null ?? "default"     // "default"
undefined ?? "default" // "default"

// Common use:
const port = config.port ?? 3000;
const name = user.name ?? "Anonymous";
```

---

## 📚 Section 6: Arrays & Strings

---

### 53. What are Arrays in JavaScript?

An **array** is an ordered list of values (any type). Arrays in JS are **objects** with numeric keys and a special `length` property.

```js
// Creating arrays
const fruits = ["apple", "banana", "cherry"];
const mixed  = [1, "hello", true, null, { id: 1 }];

// Accessing elements
fruits[0];           // "apple"
fruits[fruits.length - 1]; // "cherry" (last element)

// Modifying
fruits[1] = "mango";
fruits.push("grape");    // add to end   → returns new length
fruits.pop();            // remove from end → returns removed item
fruits.unshift("kiwi");  // add to start
fruits.shift();          // remove from start

// Common properties
fruits.length;           // number of elements
```

---

### 54. What are the Important Array Methods?

```js
const nums = [1, 2, 3, 4, 5];

// --- Non-mutating (return new array / value) ---
nums.map(n => n * 2);              // [2, 4, 6, 8, 10]
nums.filter(n => n > 2);           // [3, 4, 5]
nums.reduce((acc, n) => acc + n, 0); // 15
nums.find(n => n > 3);             // 4 (first match)
nums.findIndex(n => n > 3);        // 3 (index of first match)
nums.includes(3);                  // true
nums.some(n => n > 4);             // true (at least one)
nums.every(n => n > 0);            // true (all)
nums.slice(1, 3);                  // [2, 3] (start, end exclusive)
[...nums].reverse();               // [5, 4, 3, 2, 1]
nums.concat([6, 7]);               // [1, 2, 3, 4, 5, 6, 7]
nums.join(" - ");                  // "1 - 2 - 3 - 4 - 5"
nums.flat();                       // flattens nested arrays one level
nums.flatMap(n => [n, n * 2]);     // map + flat in one step

// --- Mutating (modify original array) ---
nums.push(6);                      // adds to end
nums.pop();                        // removes from end
nums.unshift(0);                   // adds to start
nums.shift();                      // removes from start
nums.splice(1, 2);                 // removes 2 elements at index 1
nums.splice(1, 0, 10, 20);        // inserts 10, 20 at index 1
nums.sort((a, b) => a - b);        // sorts in place (ascending)
nums.fill(0, 2, 4);                // fills index 2-3 with 0

// Array.from — create array from iterable/array-like
Array.from("hello");               // ["h", "e", "l", "l", "o"]
Array.from({ length: 3 }, (_, i) => i + 1); // [1, 2, 3]
```

---

### 55. What are Important String Methods?

Strings in JS are **immutable** — all methods return a new string.

```js
const str = "  Hello, World!  ";

// Case
str.toLowerCase();          // "  hello, world!  "
str.toUpperCase();          // "  HELLO, WORLD!  "

// Trimming
str.trim();                 // "Hello, World!"
str.trimStart();            // "Hello, World!  "
str.trimEnd();              // "  Hello, World!"

// Searching
str.includes("World");      // true
str.startsWith("  Hello");  // true
str.endsWith("!  ");        // true
str.indexOf("o");           // 5 (first occurrence)
str.lastIndexOf("o");       // 9

// Extracting
str.slice(2, 7);            // "Hello"
str.substring(2, 7);        // "Hello"
str.charAt(2);              // "H"
str[2];                     // "H"

// Replacing
str.replace("World", "JS"); // "  Hello, JS!  "
str.replaceAll("l", "L");   // replaces all occurrences

// Splitting / Joining
"a,b,c".split(",");         // ["a", "b", "c"]
["a", "b", "c"].join("-");  // "a-b-c"

// Padding & Repetition
"5".padStart(3, "0");       // "005"
"5".padEnd(3, "0");         // "500"
"ha".repeat(3);             // "hahaha"

// Template Literals (ES6)
const name = "Alice";
`Hello, ${name}!`;          // "Hello, Alice!"
`${2 + 2}`;                 // "4"
```

---

### 56. What is Type Conversion vs Type Coercion?

**Type Conversion** = **explicit** (developer intentionally converts)
**Type Coercion** = **implicit** (JS automatically converts)

```js
// --- Explicit Conversion ---
Number("42");          // 42
Number("abc");         // NaN
Number(true);          // 1
Number(false);         // 0
Number(null);          // 0
Number(undefined);     // NaN

String(42);            // "42"
String(true);          // "true"
String(null);          // "null"

Boolean(0);            // false
Boolean("");           // false
Boolean(null);         // false
Boolean(undefined);    // false
Boolean(NaN);          // false
Boolean(1);            // true
Boolean("hello");      // true
Boolean([]);           // true  ← empty array is truthy!
Boolean({});           // true  ← empty object is truthy!

parseInt("42px");      // 42 (parses until non-digit)
parseFloat("3.14abc"); // 3.14

// --- Implicit Coercion (JS does it automatically) ---
"5" + 2         // "52"  (+ triggers string concatenation)
"5" - 2         // 3     (- converts "5" to number)
"5" * "2"       // 10
true + 1        // 2
false + 1       // 1
null + 1        // 1
undefined + 1   // NaN
```

---

### 57. What are Truthy and Falsy Values?

In JavaScript, every value has a "truthiness". In boolean contexts (if, while, `||`, `&&`), values are coerced to `true` or `false`.

**Falsy values (only 8):**
```js
false
0
-0
0n           // BigInt zero
""           // empty string
null
undefined
NaN
```

**Everything else is truthy, including:**
```js
"0"          // truthy! (non-empty string)
"false"      // truthy!
[]           // truthy! (empty array)
{}           // truthy! (empty object)
-1           // truthy!
Infinity     // truthy!
```

```js
// Practical use
const name = "";
if (name) {
  console.log("Has name");
} else {
  console.log("No name"); // this runs — empty string is falsy
}

// Short-circuit evaluation
const user = null;
const display = user || "Guest";  // "Guest" (user is falsy)
const id = user && user.id;       // null (short-circuits at user)
```

---

### 58. What is the Difference Between `slice()` and `splice()`?

| Feature | `slice()` | `splice()` |
|---|---|---|
| Mutates original | ❌ No | ✅ Yes |
| Returns | New array (extracted portion) | Array of removed elements |
| Used for | Extracting / copying | Inserting, removing, replacing |

```js
const arr = [1, 2, 3, 4, 5];

// slice(start, end) — end is EXCLUSIVE, does NOT mutate
arr.slice(1, 3);    // [2, 3]
arr.slice(-2);      // [4, 5] (last 2 elements)
arr.slice();        // [1, 2, 3, 4, 5] (shallow copy)
console.log(arr);   // [1, 2, 3, 4, 5] ← unchanged

// splice(start, deleteCount, ...itemsToInsert) — MUTATES original
arr.splice(1, 2);             // removes [2, 3], arr = [1, 4, 5]
arr.splice(1, 0, 10, 20);     // inserts, arr = [1, 10, 20, 4, 5]
arr.splice(1, 1, 99);         // replaces index 1 with 99
```

---

## 🛠️ Section 7: DOM & Browser

---

### 59. What is the DOM?

The **DOM (Document Object Model)** is a programming interface that represents an HTML document as a **tree of objects**. JavaScript can read and manipulate this tree to dynamically change the page.

```
document
└── html
    ├── head
    │   └── title
    └── body
        ├── h1
        └── div
            ├── p
            └── button
```

```js
// Selecting elements
document.getElementById("myId");
document.querySelector(".myClass");         // first match (CSS selector)
document.querySelectorAll("p");             // NodeList of all <p>

// Reading / changing content
const el = document.querySelector("h1");
el.textContent;           // get text
el.textContent = "Hello"; // set text
el.innerHTML;             // get HTML (including child tags)
el.innerHTML = "<b>Bold</b>"; // set HTML (be careful of XSS!)

// Attributes
el.getAttribute("class");
el.setAttribute("data-id", "123");
el.removeAttribute("hidden");

// Styles
el.style.color = "red";
el.style.fontSize = "20px";
el.classList.add("active");
el.classList.remove("hidden");
el.classList.toggle("open");
el.classList.contains("active"); // true/false
```

---

### 60. What are DOM Events and Event Listeners?

An **event** is a signal that something happened (click, keypress, form submit, etc.).

```js
const btn = document.querySelector("#myBtn");

// Add event listener
btn.addEventListener("click", function(event) {
  console.log("Clicked!", event);
  event.preventDefault();   // prevent default behavior (e.g., form submit)
  event.stopPropagation();  // stop bubbling to parent elements
});

// Removing a listener (must reference same function)
function handleClick(e) { console.log("clicked"); }
btn.addEventListener("click", handleClick);
btn.removeEventListener("click", handleClick);

// Common events:
// click, dblclick, mouseover, mouseout, mousemove
// keydown, keyup, keypress
// submit, change, input, focus, blur
// load, DOMContentLoaded, resize, scroll

// DOMContentLoaded — DOM ready (before images load)
document.addEventListener("DOMContentLoaded", () => {
  console.log("DOM is ready!");
});
```

---

### 61. What is Event Bubbling and Capturing?

When an event occurs on a nested element, it **propagates** through the DOM in two phases:

1. **Capture phase** — event travels DOWN from document → target
2. **Bubble phase** — event travels UP from target → document (default)

```html
<div id="outer">
  <button id="inner">Click me</button>
</div>
```

```js
document.getElementById("outer").addEventListener("click", () => {
  console.log("Outer clicked");
});
document.getElementById("inner").addEventListener("click", () => {
  console.log("Inner clicked");
});

// Click button → Output:
// "Inner clicked"
// "Outer clicked"  ← bubbles up

// To listen in capture phase, pass true as 3rd arg:
document.getElementById("outer").addEventListener("click", () => {
  console.log("Outer capture");
}, true);

// stop bubbling:
btn.addEventListener("click", (e) => {
  e.stopPropagation(); // won't reach outer
});
```

---

## ⚠️ Section 8: Error Handling

---

### 62. What is `try`, `catch`, `finally`?

```js
try {
  // code that might throw
  const result = riskyOperation();
  console.log(result);
} catch (error) {
  // runs if try block throws
  console.error("Error:", error.message);
  console.error("Stack:", error.stack);
} finally {
  // ALWAYS runs — used for cleanup
  closeConnection();
}

// Throwing custom errors
function divide(a, b) {
  if (b === 0) throw new Error("Cannot divide by zero");
  return a / b;
}

try {
  divide(10, 0);
} catch (e) {
  console.log(e.message); // "Cannot divide by zero"
  console.log(e instanceof Error); // true
}
```

---

### 63. What are the Types of Errors in JavaScript?

```js
// 1. SyntaxError — invalid syntax (caught at parse time)
// const x = ;  ← SyntaxError

// 2. ReferenceError — using an undeclared variable
undeclaredVar; // ReferenceError: undeclaredVar is not defined

// 3. TypeError — wrong type operation
null.property; // TypeError: Cannot read property of null
const n = 42;
n();           // TypeError: n is not a function

// 4. RangeError — value out of allowed range
new Array(-1); // RangeError: Invalid array length
(1.5).toFixed(200); // RangeError

// 5. URIError — bad URI encoding
decodeURIComponent("%"); // URIError

// Custom Error types
class ValidationError extends Error {
  constructor(message, field) {
    super(message);
    this.name = "ValidationError";
    this.field = field;
  }
}

try {
  throw new ValidationError("Email is required", "email");
} catch (e) {
  if (e instanceof ValidationError) {
    console.log(e.field);   // "email"
    console.log(e.message); // "Email is required"
  }
}
```

---

## 🧱 Section 9: ES6+ Features

---

### 64. What are ES6 Classes?

ES6 `class` is **syntactic sugar** over JS's existing prototypal inheritance. It doesn't introduce a new OOP model.

```js
class Animal {
  // Constructor — called when 'new Animal()' is used
  constructor(name, sound) {
    this.name = name;
    this.sound = sound;
  }

  // Instance method
  speak() {
    return `${this.name} says ${this.sound}!`;
  }

  // Static method — called on class, not instance
  static create(name, sound) {
    return new Animal(name, sound);
  }

  // Getter
  get info() {
    return `${this.name} (${this.sound})`;
  }
}

const cat = new Animal("Cat", "meow");
cat.speak();          // "Cat says meow!"
cat.info;             // "Cat (meow)"
Animal.create("Dog", "woof"); // static

// Inheritance
class Dog extends Animal {
  constructor(name, breed) {
    super(name, "woof"); // must call super() first!
    this.breed = breed;
  }

  // Override parent method
  speak() {
    return super.speak() + ` (${this.breed})`;
  }
}

const d = new Dog("Rex", "Labrador");
d.speak();           // "Rex says woof! (Labrador)"
d instanceof Dog;    // true
d instanceof Animal; // true
```

---

### 65. What are Template Literals?

Template literals use **backticks** `` ` `` and allow embedded expressions and multi-line strings.

```js
const name = "Alice";
const age = 25;

// String interpolation
const msg = `Hello, ${name}! You are ${age} years old.`;

// Expressions
`${2 + 2}`;          // "4"
`${age >= 18 ? "Adult" : "Minor"}`;  // "Adult"
`${name.toUpperCase()}`;             // "ALICE"

// Multi-line strings (no \n needed)
const html = `
  <div>
    <h1>${name}</h1>
    <p>Age: ${age}</p>
  </div>
`;

// Tagged templates (advanced)
function highlight(strings, ...values) {
  return strings.map((str, i) =>
    str + (values[i] ? `<b>${values[i]}</b>` : "")
  ).join("");
}
highlight`Hello ${name}, you are ${age} years old.`;
// "Hello <b>Alice</b>, you are <b>25</b> years old."
```

---

### 66. What are Modules in JavaScript (`import` / `export`)?

ES6 introduced native modules to split code across files with `import`/`export`.

```js
// math.js — Named exports
export const PI = 3.14159;
export function add(a, b) { return a + b; }
export function multiply(a, b) { return a * b; }

// Default export (one per file)
export default class Calculator { ... }

// -----------------------------------------------

// main.js — Importing
import Calculator from "./math.js";          // default import
import { PI, add } from "./math.js";         // named imports
import { add as sum } from "./math.js";      // alias
import * as Math from "./math.js";           // import all as namespace

// Dynamic import (lazy loading)
const module = await import("./heavy.js");
```

**CommonJS (Node.js old style):**
```js
// Exporting
module.exports = { add, multiply };
module.exports.PI = 3.14;

// Importing
const { add, multiply } = require("./math");
const math = require("./math");
```

---

### 67. What is Destructuring Assignment in Arrays?

```js
// Basic array destructuring
const [a, b, c] = [1, 2, 3];
console.log(a, b, c); // 1 2 3

// Skip elements
const [first, , third] = [1, 2, 3];
console.log(first, third); // 1 3

// Default values
const [x = 10, y = 20] = [5];
console.log(x, y); // 5 20

// Rest in destructuring
const [head, ...tail] = [1, 2, 3, 4, 5];
console.log(head); // 1
console.log(tail); // [2, 3, 4, 5]

// Swap without temp variable
let p = 1, q = 2;
[p, q] = [q, p];
console.log(p, q); // 2 1

// From function return
function getCoords() { return [10, 20]; }
const [lat, lng] = getCoords();

// Nested array destructuring
const [[a1, a2], [b1, b2]] = [[1, 2], [3, 4]];
```

---

### 68. What is the Ternary Operator?

The **ternary operator** is a compact `if...else` shorthand: `condition ? exprIfTrue : exprIfFalse`

```js
// if...else version
let message;
if (age >= 18) {
  message = "Adult";
} else {
  message = "Minor";
}

// Ternary version
const message = age >= 18 ? "Adult" : "Minor";

// Nested ternary (use sparingly — hurts readability)
const grade = score >= 90 ? "A"
            : score >= 80 ? "B"
            : score >= 70 ? "C"
            : "F";

// In JSX (React)
const element = isLoggedIn ? <Dashboard /> : <Login />;
```

---

### 69. What is Short-Circuit Evaluation?

`&&` and `||` don't just return booleans — they return one of their **operand values** based on truthiness.

```js
// || (OR) — returns first TRUTHY value, or last value if all falsy
"hello" || "world"    // "hello" (first truthy)
null || "default"     // "default"
0 || "" || "fallback" // "fallback"
null || undefined     // undefined (last value, both falsy)

// && (AND) — returns first FALSY value, or last value if all truthy
"hello" && "world"    // "world" (all truthy, returns last)
null && "world"       // null (first falsy, short-circuits)
user && user.name     // user.name (safe navigation pattern)

// Practical patterns
const name = user.name || "Anonymous";  // default value
user && sendEmail(user);                // run only if truthy
isAdmin && renderAdminPanel();          // conditional render (React)

// ?? is better for defaults to avoid issues with 0 or ""
const count = data.count ?? 0;
```

---

### 70. What is Recursion?

**Recursion** is when a function **calls itself** until a **base case** is reached.

```js
// Factorial — n! = n * (n-1) * ... * 1
function factorial(n) {
  if (n <= 1) return 1;  // base case
  return n * factorial(n - 1); // recursive case
}
factorial(5); // 5 * 4 * 3 * 2 * 1 = 120

// Fibonacci
function fib(n) {
  if (n <= 1) return n;
  return fib(n - 1) + fib(n - 2);
}
fib(6); // 8

// Countdown
function countdown(n) {
  if (n < 0) return;
  console.log(n);
  countdown(n - 1);
}
countdown(3); // 3, 2, 1, 0

// Flatten nested array recursively
function flatten(arr) {
  return arr.reduce((acc, item) =>
    Array.isArray(item) ? [...acc, ...flatten(item)] : [...acc, item],
  []);
}
flatten([1, [2, [3, [4]]]]); // [1, 2, 3, 4]
```

> ⚠️ Always have a **base case** — without it you get infinite recursion and a stack overflow!

---

### 71. What is JSON and how do you work with it?

**JSON (JavaScript Object Notation)** is a lightweight **text format** for storing and transmitting data. It looks like JS object syntax but is always a string.

```js
// JS Object → JSON string  (Serialization)
const user = { name: "Alice", age: 25, active: true };
const jsonStr = JSON.stringify(user);
// '{"name":"Alice","age":25,"active":true}'

// Pretty formatting
JSON.stringify(user, null, 2);
// {
//   "name": "Alice",
//   "age": 25,
//   "active": true
// }

// JSON string → JS Object  (Parsing / Deserialization)
const parsed = JSON.parse(jsonStr);
parsed.name; // "Alice"

// Error handling (invalid JSON throws SyntaxError)
try {
  JSON.parse("invalid json");
} catch(e) {
  console.error("Invalid JSON:", e.message);
}

// JSON rules:
// ✅ Strings must use double quotes: "key"
// ✅ Values: string, number, boolean, null, array, object
// ❌ No: undefined, functions, symbols, Date (converted to string)
// ❌ No trailing commas
```

---

### 72. What is the `new` Keyword?

The `new` keyword is used to **create an instance** of a constructor function or class. It:
1. Creates a new empty object `{}`
2. Sets `this` to point to that new object
3. Runs the constructor function
4. Sets `__proto__` to the constructor's `prototype`
5. Returns `this` (unless constructor returns another object)

```js
function Person(name, age) {
  // 'this' = new empty object
  this.name = name;
  this.age = age;
  // implicitly returns 'this'
}

const alice = new Person("Alice", 25);
alice.name;              // "Alice"
alice instanceof Person; // true

// Without 'new' — THIS would be global/undefined!
const oops = Person("Bob", 30); // 'this' = window!
oops; // undefined

// Same with classes (class enforces 'new' usage)
class Car {
  constructor(brand) { this.brand = brand; }
}
const myCar = new Car("Toyota");
// Car("BMW"); // TypeError: Class must be called with 'new'
```

---

### 73. What is `Array.from()` and `Array.of()`?

```js
// Array.from() — creates an array from any iterable or array-like object
Array.from("hello");           // ["h", "e", "l", "l", "o"]
Array.from(new Set([1, 2, 3])); // [1, 2, 3]
Array.from(new Map([["a", 1]])); // [["a", 1]]
Array.from({ length: 5 }, (_, i) => i * 2); // [0, 2, 4, 6, 8]
Array.from(document.querySelectorAll("p")); // real Array from NodeList

// Array.of() — creates an array from arguments (unlike 'new Array')
Array.of(7);        // [7]      ← one-element array
new Array(7);       // [,,,,,,, ] ← 7 empty slots (confusing!)
Array.of(1, 2, 3);  // [1, 2, 3]
new Array(1, 2, 3); // [1, 2, 3] (same here, but Array.of is safer)
```

---

### 74. What is Chaining in JavaScript?

**Method chaining** is calling multiple methods on an object one after another because each method returns the object (or a new one).

```js
// Array chaining
const result = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
  .filter(n => n % 2 === 0)   // [2, 4, 6, 8, 10]
  .map(n => n * n)             // [4, 16, 36, 64, 100]
  .reduce((acc, n) => acc + n, 0); // 220

// String chaining
"  Hello World  "
  .trim()
  .toLowerCase()
  .replace("world", "JS")
  .split(" ")
  .join("-"); // "hello-js"

// Building your own chainable API
class QueryBuilder {
  constructor() { this.query = "SELECT *"; }
  from(table) { this.query += ` FROM ${table}`; return this; }
  where(cond) { this.query += ` WHERE ${cond}`; return this; }
  limit(n)    { this.query += ` LIMIT ${n}`; return this; }
  build()     { return this.query; }
}

new QueryBuilder()
  .from("users")
  .where("age > 18")
  .limit(10)
  .build();
// "SELECT * FROM users WHERE age > 18 LIMIT 10"
```

---

### 75. What is `Object.keys()`, `Object.values()`, and `Object.entries()`?

```js
const user = { name: "Alice", age: 25, city: "Mumbai" };

// Object.keys() — returns array of own enumerable property KEYS
Object.keys(user);    // ["name", "age", "city"]

// Object.values() — returns array of own enumerable property VALUES
Object.values(user);  // ["Alice", 25, "Mumbai"]

// Object.entries() — returns array of [key, value] pairs
Object.entries(user); // [["name", "Alice"], ["age", 25], ["city", "Mumbai"]]

// Practical uses
Object.keys(user).forEach(key => console.log(key, user[key]));

// Convert object to Map
const map = new Map(Object.entries(user));

// Filter object properties
const filtered = Object.fromEntries(
  Object.entries(user).filter(([key, val]) => typeof val === "string")
);
// { name: "Alice", city: "Mumbai" }

// Clone & transform
const doubled = Object.fromEntries(
  Object.entries({ a: 1, b: 2 }).map(([k, v]) => [k, v * 2])
);
// { a: 2, b: 4 }
```

---

### 76. What is the difference between `null` check and `undefined` check?

```js
let x;           // undefined — declared but not assigned
let y = null;    // null — explicitly set to empty

// Check for undefined
x === undefined      // true ✅
typeof x === "undefined" // true ✅ (safe even if x not declared)

// Check for null
y === null           // true ✅

// Check for EITHER null or undefined (nullish check)
x == null            // true (loose equality catches both)
y == null            // true

// Practical patterns
function getUser(id) {
  if (id == null) throw new Error("id is required"); // catches both null & undefined
}

// Nullish coalescing
const name = user.name ?? "Anonymous"; // only if null or undefined
const name2 = user.name || "Anonymous"; // also triggers on "" and 0
```

---

### 77. What are Getter and Setter in ES6 Classes?

```js
class Circle {
  constructor(radius) {
    this._radius = radius; // _ prefix = "private by convention"
  }

  // Getter — access like a property, not a method call
  get radius() { return this._radius; }
  get diameter() { return this._radius * 2; }
  get area() { return Math.PI * this._radius ** 2; }

  // Setter — validate before setting
  set radius(value) {
    if (typeof value !== "number" || value <= 0) {
      throw new Error("Radius must be a positive number");
    }
    this._radius = value;
  }
}

const c = new Circle(5);
c.radius;        // 5  (calls getter, no parentheses!)
c.diameter;      // 10
c.area;          // 78.539...
c.radius = 10;   // calls setter (validates)
c.radius = -1;   // throws Error!
```

---

### 78. What is `typeof` vs `instanceof`?

```js
// typeof — returns a string describing the type of a primitive
typeof "hello"       // "string"
typeof 42            // "number"
typeof true          // "boolean"
typeof undefined     // "undefined"
typeof null          // "object" ← bug!
typeof []            // "object"
typeof {}            // "object"
typeof function(){}  // "function"

// instanceof — checks if an object is an instance of a class/constructor
// Checks the prototype chain
[] instanceof Array    // true
[] instanceof Object   // true (Array inherits from Object)
{} instanceof Object   // true

function Person(name) { this.name = name; }
const p = new Person("Alice");
p instanceof Person    // true
p instanceof Object    // true

// typeof is best for primitives
// instanceof is best for objects/classes

// Reliable type checking using Object.prototype.toString:
Object.prototype.toString.call([]);       // "[object Array]"
Object.prototype.toString.call({});       // "[object Object]"
Object.prototype.toString.call(null);     // "[object Null]"
Object.prototype.toString.call(new Date); // "[object Date]"
```

---

### 79. What are Common Interview Coding Patterns?

#### Reverse a String:
```js
"hello".split("").reverse().join(""); // "olleh"
[..."hello"].reverse().join("");      // same
```

#### Check Palindrome:
```js
function isPalindrome(str) {
  const clean = str.toLowerCase().replace(/[^a-z0-9]/g, "");
  return clean === clean.split("").reverse().join("");
}
isPalindrome("racecar"); // true
isPalindrome("hello");   // false
```

#### Find Duplicates in Array:
```js
function findDuplicates(arr) {
  return arr.filter((item, index) => arr.indexOf(item) !== index);
}
findDuplicates([1, 2, 2, 3, 3, 4]); // [2, 3]
```

#### Flatten Array:
```js
[1, [2, [3, [4]]]].flat(Infinity); // [1, 2, 3, 4]
```

#### Count Character Occurrences:
```js
function charCount(str) {
  return [...str].reduce((acc, char) => {
    acc[char] = (acc[char] || 0) + 1;
    return acc;
  }, {});
}
charCount("hello"); // { h: 1, e: 1, l: 2, o: 1 }
```

#### Remove Duplicates from Array:
```js
const unique = [...new Set([1, 2, 2, 3, 3])]; // [1, 2, 3]
```

---

### 80. What are the Key Coding Best Practices in JavaScript?

```js
// ✅ 1. Use const by default, let when needed, never var
const MAX = 100;
let count = 0;

// ✅ 2. Use strict equality (===)
if (value === null) { }

// ✅ 3. Use optional chaining for safe access
const city = user?.address?.city ?? "Unknown";

// ✅ 4. Prefer arrow functions for callbacks
const doubled = nums.map(n => n * 2);

// ✅ 5. Avoid modifying function arguments
function process(arr) {
  const copy = [...arr]; // work on a copy
  copy.sort();
  return copy;
}

// ✅ 6. Handle errors gracefully
async function fetchData() {
  try {
    const data = await getdata();
    return data;
  } catch(err) {
    console.error(err);
    return null;
  }
}

// ✅ 7. Use meaningful variable names
const u = getUser();   // ❌ vague
const currentUser = getUser(); // ✅ clear

// ✅ 8. Avoid deeply nested code — early return pattern
function processOrder(order) {
  if (!order) return null;             // early return
  if (!order.items.length) return [];  // early return
  return order.items.map(transform);
}

// ✅ 9. Use template literals over concatenation
const msg = `Hello, ${name}!`;  // ✅
const msg2 = "Hello, " + name + "!"; // ❌

// ✅ 10. Use destructuring
const { name, age } = user;          // ✅
const name2 = user.name;             // ❌ verbose
```

---

*End of JavaScript Fundamentals Reference*

---

### 81. What is the `void` Operator?

The `void` operator evaluates the given expression and then returns `undefined`. It's often used in HTML to prevent a link from navigating.

```js
// Usage in HTML
// <a href="javascript:void(0)">Do nothing</a>

const result = void (2 + 2); 
console.log(result); // undefined
```

---

### 82. What is the `delete` Operator?

The `delete` operator removes a property from an object. If the property's value is an object and there are no more references to it, it will eventually be garbage collected.

```js
const user = { name: "Alice", age: 25 };
delete user.age; 
console.log(user); // { name: "Alice" }

// Note: It doesn't affect variables (let, const, var)
let x = 10;
delete x; // false (returns false in non-strict, throws in strict)
```

---

### 83. What is the `in` Operator vs `hasOwnProperty`?

-   **`in` operator:** Checks if a property exists in the object **or its prototype chain**.
-   **`hasOwnProperty()`:** Checks if a property exists **only as a direct property** of the object (ignores the prototype chain).

```js
const user = { name: "Alice" };

console.log("name" in user);           // true
console.log("toString" in user);       // true (from Object.prototype)

console.log(user.hasOwnProperty("name"));     // true
console.log(user.hasOwnProperty("toString")); // false
```

---

### 84. What are Bitwise Operators in JavaScript?

Bitwise operators treat their operands as a sequence of 32 bits (zeros and ones) rather than as decimal numbers.
-   `&` (AND), `|` (OR), `^` (XOR), `~` (NOT), `<<` (Left shift), `>>` (Right shift).

```js
// 5 = 0101
// 1 = 0001
console.log(5 & 1); // 0001 → 1
console.log(5 | 1); // 0101 → 5
```
*Note: Rarely used in web development but common in low-performance-sensitive graphics or coding interviews.*

---

### 85. What is `BigInt`?

`BigInt` is a primitive type introduced in ES2020 used to represent integers with arbitrary precision — beyond the `Number.MAX_SAFE_INTEGER` (2^53 - 1).

```js
const big = 123456789012345678901234567890n; // suffix 'n'
const another = BigInt("123456789012345678901234567890");

console.log(big + 1n); // works!
// console.log(big + 1); // TypeError: Cannot mix BigInt and other types
```

---

### 86. What is the `at()` Method for Arrays and Strings?

`at()` takes an integer value and returns the item at that index. It allows for **negative integers** to count back from the last item.

```js
const arr = [10, 20, 30];
console.log(arr.at(1));  // 20
console.log(arr.at(-1)); // 30 (instead of arr[arr.length - 1])

const str = "Hello";
console.log(str.at(-2)); // "l"
```

---

### 87. What is `AbortController`?

The `AbortController` interface allows you to abort one or more Web requests (like `fetch`) as and when desired.

```js
const controller = new AbortController();
const signal = controller.signal;

fetch('/api/data', { signal })
  .then(res => res.json())
  .catch(err => {
    if (err.name === 'AbortError') console.log('Fetch aborted');
  });

// Cancel the request
controller.abort();
```

---

### 88. What is `IntersectionObserver` basics?

It provides a way to asynchronously observe changes in the intersection of a target element with an ancestor element or the browser viewport. Common for **Lazy Loading Images** or **Infinite Scroll**.

```js
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      console.log("Element is visible!");
    }
  });
});

observer.observe(document.querySelector("#myElement"));
```

---

### 89. What is the `Intl` Object?

The `Intl` object provides language-sensitive string comparison, number formatting, and date/time formatting.

```js
// Currency formatting
const formatter = new Intl.NumberFormat('en-US', {
  style: 'currency',
  currency: 'USD',
});
console.log(formatter.format(1000)); // "$1,000.00"

// Relative time formatting
const rtf = new Intl.RelativeTimeFormat('en', { numeric: 'auto' });
console.log(rtf.format(-1, 'day')); // "yesterday"
```

---

### 90. What is `crypto.getRandomValues()`?

It's a way to generate **cryptographically strong** random values, better than `Math.random()` for security cases like generating IDs or passwords.

```js
const array = new Uint32Array(10);
window.crypto.getRandomValues(array);
console.log(array);
```

---

### 91. What is the difference between `NaN` and `Number.NaN`?

They are essentially the same. `Number.NaN` is a property of the `Number` object, while `NaN` is a global property. Both represent "Not-a-Number".

```js
NaN === Number.NaN; // false (NaN is never equal to anything, even itself)
Object.is(NaN, Number.NaN); // true
```

---

### 92. What are "Labeled Statements"?

Labeled statements are used with `break` or `continue` to jump out of or continue a specific loop when you have **nested loops**.

```js
outerLoop: for (let i = 0; i < 3; i++) {
  for (let j = 0; j < 3; j++) {
    if (i === 1 && j === 1) break outerLoop; 
    console.log(i, j);
  }
}
```

---

### 93. What is the difference between `Object.entries()` and `Object.fromEntries()`?

-   **`Object.entries(obj)`:** Converts an object to an array of `[key, value]` pairs.
-   **`Object.fromEntries(pairs)`:** Converts an array of `[key, value]` pairs back into an object.

```js
const obj = { a: 1, b: 2 };
const entries = Object.entries(obj); // [["a", 1], ["b", 2]]

const newObj = Object.fromEntries(entries); // { a: 1, b: 2 }
```

---

### 94. What is `structuredClone()`?

A modern built-in function to create a **deep copy** of a value (including nested objects/arrays), replacing the old `JSON.parse(JSON.stringify(obj))` hack.

```js
const original = { a: 1, b: { c: 2 } };
const copy = structuredClone(original);

copy.b.c = 3;
console.log(original.b.c); // 2 (unchanged! deep copy)
```

---

### 100. Best Practices for Clean Code in JavaScript?

1.  **Use Meaningful Names:** `getUserData()` instead of `getData()`.
2.  **Keep Functions Small:** A function should do one thing well.
3.  **Use ES6+ Features:** Prefer spread, destructuring, and template literals for readability.
4.  **Avoid Global Variables:** Use modules or local scope.
5.  **Write Comments Why, Not What:** Code should explain *what* it does; comments explain *why*.
6.  **Handle Errors Gracefully:** Always use try/catch for risky operations.
7.  **Format Consistently:** Use tools like Prettier and ESLint.

