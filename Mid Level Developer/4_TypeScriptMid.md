# TypeScript for Backend – Mid-Level Developer Interview Reference

---

## 🔷 Section 1: Advanced Type Mechanics & Design Patterns

---

### 1. Why use TypeScript for Backend Development (Architectural Perspective)?

**Definition:** Beyond "finding bugs", TypeScript is a tool for **Domain Driven Design (DDD)** and **Contract-First Development**.

**Key Backend Advantages:**
1.  **Contract Definition:** Interfaces serve as the "Single Source of Truth" for API requests, database models, and microservice communication.
2.  **Refactoring at Scale:** Safely rename fields (e.g., `user_id` to `userId`) across a codebase of 100k+ lines. Without TS, this is a dangerous "Search & Replace" game.
3.  **Strict Null Checks:** Eliminates 90% of `Cannot read property of undefined` errors by forcing you to handle `null | undefined` explicitly.
4.  **Documentation as Types:** The code becomes self-documenting. A function signature `(id: UserId) => Promise<User>` tells you everything you need to know without reading the function body.

---

### 2. Interface vs Type Alias: The Mid-Level Nuance

| Feature | Interface | Type Alias |
|---|---|---|
| **Declaration Merging** | **Yes.** Can define the same interface twice; fields are merged. (Good for plugins/extending libraries). | **No.** Name must be unique. |
| **Complex Types** | No. Cannot represent Unions, Tuples, or Primitives directly. | **Yes.** Can represent `Union`, `Intersection`, `Tuple`, etc. |
| **Extension Syntax** | `interface A extends B` | `type A = B & C` |
| **Performance** | **Faster.** The compiler caches interface relationships better during checks. | Slightly slower for very complex recursions. |

**Practical Rule:** Use **Interfaces** for object shapes and classes (resembles public contracts). Use **Types** for internal logic, unions, and mapped types.

---

### 3. What are "Branded Types" (Nominal Typing)?

**Problem:** TypeScript is **Structural**. If two types have the same shape, they are treated as identical. 
`function deleteUser(id: string)` — You could accidentally pass a `postId` instead of a `userId` because both are strings.

**The Solution:**
```ts
type Brand<K, T> = K & { __brand: T };

type UserId = Brand<string, "UserId">;
type PostId = Brand<string, "PostId">;

function deleteUser(id: UserId) { ... }

const pid = "abc" as PostId;
deleteUser(pid); // ❌ Error: Argument is not assignable to type UserId.
```

---

### 4. Template Literal Types (TS 4.1+)

**Definition:** Allows you to build new string literal types by manipulating existing ones.

**Real-world Use Case (Event Handlers):**
```ts
type EventName = "click" | "hover" | "scroll";
type CallbackName = `on${Capitalize<EventName>}`; 
// Result: "onClick" | "onHover" | "onScroll"

// API Routes
type Resource = "User" | "Order";
type Route = `/api/v1/${Lowercase<Resource>}s`;
// Result: "/api/v1/users" | "/api/v1/orders"
```

---

### 5. Conditional Types & the `infer` Keyword

**Definition:** Conditional types allow you to define types that change based on a condition (`T extends U ? X : Y`). 

**The `infer` keyword:** Used inside the `extends` clause to "extract" a type.

**Example (Unpacking Promise value):**
```ts
type Unpack<T> = T extends Promise<infer U> ? U : T;

type User = { name: string };
type Resolved = Unpack<Promise<User>>; // Result: User
```

---

### 6. "Exhaustive Checking" with the `never` type

**Problem:** You have a union `Action = Create | Delete`. You write a switch. Later, someone adds `Update` to the union, but forgets to update the switch.

**The Solution:**
```ts
function handleAction(action: Action) {
  switch (action.type) {
    case "create": return "Creating...";
    case "delete": return "Deleting...";
    default:
      // If the union is exhausted, 'action' will be typed as 'never'
      const _check: never = action; 
      // ❌ Compile error here if 'update' was added to Action!
      return _check;
  }
}
```

---

### 7. What is the `satisfies` Operator (TS 4.9)?

**Problem:** If you type a variable as `Record<string, string>`, you lose the specific keys' information.

**Solution:** `satisfies` checks that a value matches a type **without** changing the value's type to that broader type.
```ts
const colors = {
  red: "#ff0000",
  blue: [0, 0, 255],
} satisfies Record<string, string | number[]>;

colors.red.toUpperCase(); // ✅ Works! TS knows it's a string.
colors.blue.map(x => x);  // ✅ Works! TS knows it's an array.
```

---

### 8. Utility Types Deep-Dive: `Pick`, `Omit`, `Awaited`

-   **`Pick<T, K>`**: Extracts specific fields for a "view" (e.g., `UserSummary`).
-   **`Omit<T, K>`**: Removes sensitive fields (e.g., `Omit<User, 'password'>`).
-   **`Awaited<T>`**: Handles nested promises. If `T` is `Promise<Promise<string>>`, `Awaited<T>` is `string`. Essential for typing generic async wrappers.

---

### 9. Covariance and Contravariance (Advanced)

This relates to how types behave in function arguments and return values.
-   **Return types are Covariant:** You can return a "more specific" type than requested.
-   **Argument types are Contravariant:** You can accept a "more general" type in a callback.

**Why it matters:** Understanding this prevents "Type 'X' is not assignable to type 'Y'" errors when dealing with Higher-Order Functions or complex class hierarchies.

---

### 10. `abstract` Class vs `interface` for Backend Architecture

-   **Interface:** Best for defining **External Contracts** (API response, DB models). They disappear at runtime (zero overhead).
-   **Abstract Class:** Best for defining **Internal Frameworks**. For example, an `AbstractRepository` that contains shared `save()` and `delete()` logic that you want every specific repository (UserRepo, PostRepo) to inherit and reuse.

---

### 11. Const Assertions (`as const`)

**Definition:** It tells the compiler to treat the entire object as **Read Only** and to infer the **literal types** instead of general types.

**Example:**
```ts
const COLORS = {
  RED: "#FF0000",
  BLUE: "#0000FF",
} as const;

// COLORS.RED is "#FF0000" (literal), not string.
// COLORS.RED = "green"; ❌ Error! Ready-only.
```

---

### 12. Recursive Types: Building a JSON Type

**Problem:** How do you define a type for a valid JSON object?

**Solution:**
```ts
type JSONValue = 
  | string 
  | number 
  | boolean 
  | null 
  | { [key: string]: JSONValue } 
  | JSONValue[];
```

---

### 13. DeepPartial: Recursively making fields optional

**Problem:** `Partial<T>` only works at the first level. What if your object has nested objects?

**Solution:**
```ts
type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P];
};
```

---

### 14. What are `in` and `is` (Type Predicates)?

**Mechanism:** Used to create a custom type guard that provides 100% type safety within a block.
```ts
function isUser(obj: any): obj is User {
  return "name" in obj && "email" in obj;
}

if (isUser(data)) {
  console.log(data.name); // ✅ TS knows 'data' is User
}
```

---

### 15. The `unknown` type vs `any`

A mid-level developer should never use `any`!
-   **`any`**: Turns off type checking. 
-   **`unknown`**: Tells TS "I don't know what this is yet". You are **Forced** to perform an `if` check or a type assertion before you can use it. This is significantly safer for API responses.

---

### 16. Function Overloading in TS

Unlike Java/C#, TS only has **ONE** implementation, but multiple signatures. This is used to handle different argument combinations safely.
```ts
function makeDate(timestamp: number): Date;
function makeDate(m: number, d: number, y: number): Date;
function makeDate(mOrTimestamp: number, d?: number, y?: number): Date {
  if (d !== undefined && y !== undefined) {
    return new Date(y, mOrTimestamp, d);
  } else {
    return new Date(mOrTimestamp);
  }
}
```

---

### 17. Discriminated Unions: Why "Literals" are key?

Instead of `if (data.isSuccess)`, use `if (data.type === 'SUCCESS')`.
-   **Why?** A Boolean only gives you 2 states. A string Literal discriminant allows for N states (Success, Error, Pending, Retrying) and works perfectly with `switch` statements for Exhaustive Checking.

---

### 18. Module Augmentation: Extending Express Request

**Problem:** How do you tell TS that `req.user` exists after your middleware runs?

**Solution:**
```ts
declare global {
  namespace Express {
    interface Request {
      user?: { id: string; role: string };
    }
  }
}
```

---

### 19. Key Remapping in Mapped Types (TS 4.1+)

**Definition:** You can use a Template Literal to change the keys of an object while mapping.
```ts
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

interface User { name: string; age: number; }
type UserGetters = Getters<User>; 
// Result: { getName: () => string; getAge: () => number; }
```

---

### 20. The `unique symbol` type

**Definition:** Used for creating globally unique object properties that are hidden from normal iteration.
-   **Mid-level Use Case:** Creating "Private" properties in a class or object that are truly hidden from third-party libraries (unless you explicitly export the symbol).

---

### 21. Improving Errors with `const` Type Parameters (TS 5.0)

**Definition:** A new way to tell TypeScript to infer the most specific "Const" type possible for an argument without the user having to add `as const` manually.
```ts
function getNames<const T extends string[]>(names: T) { ... }

getNames(["Alice", "Bob"]); // TS knows T is ["Alice", "Bob"], not string[]
```

---

### 22. Truthiness Narrowing Pitfalls

**Problem:** `if (val)` treats `0`, `""`, and `null` all as false.
-   **Mid-level tip:** Always be explicit. `if (val !== undefined)` or `if (typeof val === 'number')`. This prevents bugs where a valid `0` is accidentally treated as "Missing Data".

---

### 23. Decorators vs HOF (Higher Order Functions)

-   **Decorators:** Add metadata or behavior to classes/methods (`@Post`, `@Inject`). Preferred in frameworks like NestJS for "Clean Code".
-   **HOFs:** Used in functional programming (`withLogging(handler)`). 
-   **Difference:** Decorators are a stage 3 TC39 proposal in JS, while HOFs are standard JS functions.

---

### 24. TypeScript Mono-repo: Project References

**Definition:** The `tsconfig.json` `references` field allows you to split a large project into smaller, independent TS pieces.
-   **Benefit:** Faster builds (only the changed piece is re-checked) and strict boundaries (Sub-project A cannot import from Sub-project B unless defined).

---

### 25. The `Awaited` Type (TS 4.5+)

**Problem:** You have a function that returns `Promise<string>`. You want to get `string`.
**Solution:** `Awaited<ReturnType<typeof func>>`. 
-   **Internals:** It handles nested promises recursively, which is why it's safer than just `infer U` in a conditional type.

---

*Expansion Complete*
