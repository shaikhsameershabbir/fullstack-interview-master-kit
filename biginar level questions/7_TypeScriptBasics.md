# TypeScript Basics – Complete Beginner Interview Reference

---

## 🟦 Section 1: Core Fundamentals

---

### 1. What is TypeScript?

**Definition:** **TypeScript** is an **open-source programming language** developed by Microsoft. It is a **strict syntactical superset of JavaScript**, which means any valid JavaScript code is also valid TypeScript. Its primary feature is adding **optional static typing** to the language.

**Key Features:**
-   **Static Typing:** Catch errors at compile-time rather than runtime.
-   **Superset:** Works with existing JS libraries and code.
-   **Readable & Maintainable:** Explicit types make code easier to understand in large teams.
-   **Advanced Tooling:** Better Autocomplete, Refactoring, and Navigation in IDEs (like VS Code).

**How it works:**
TypeScript cannot be run directly by browsers or Node.js. It must be **compiled (transpiled)** into plain JavaScript using the TypeScript Compiler (`tsc`).

---

### 2. What is the difference between TypeScript and JavaScript?

| Feature | JavaScript | TypeScript |
|---|---|---|
| **Type System** | Dynamic (types known at runtime) | Static (types known at compile-time) |
| **Error Detection** | Runtime | Compile-time (mostly) |
| **Learning Curve** | Easy to start | Requires learning types/interfaces |
| **Browser Support** | Native support | Must be compiled to JS |
| **Tooling** | Minimal | Rich (Auto-complete, Refactoring) |

---

### 3. What are the Basic Types in TypeScript?

TypeScript supports all JS types plus a few more:
1.  **`string`**: Textual data. `let name: string = "Alice";`
2.  **`number`**: Numeric values (integers and floats). `let age: number = 25;`
3.  **`boolean`**: True/False. `let idDone: boolean = true;`
4.  **`any`**: Opt-out of type checking. Use sparingly!
5.  **`unknown`**: Like `any`, but safer. You must check the type before using it.
6.  **`void`**: Used for functions that return nothing.
7.  **`never`**: Used for values that never occur (e.g., a function that always throws an error).
8.  **`null` & `undefined`**: Correspond to JS null/undefined.

---

### 4. What is Type Inference?

**Definition:** TypeScript is smart enough to **guess (infer)** the type of a variable based on the value you assign to it, even if you don't explicitly declare it.

**Example:**
```ts
let message = "Hello"; // TS infers message is a 'string'
message = 42;         // Error: Type 'number' is not assignable to 'string'
```

---

### 5. What are Union Types?

**Definition:** A Union type allows a value to be **one of several types**. You use the pipe (`|`) symbol to separate them.

**Example:**
```ts
let id: string | number;
id = 101;     // OK
id = "A101";  // OK
id = true;    // Error: Type 'boolean' is not assignable
```

---

### 6. What is a Type Alias?

**Definition:** A **Type Alias** is a way to create a **new name** for a type. It helps in making complex types more readable and reusable.

**Example:**
```ts
type UserID = string | number;
type User = {
  id: UserID;
  name: string;
};

const user: User = { id: 1, name: "Alice" };
```

---

### 7. What is an Interface?

**Definition:** An **Interface** is a way to define the **shape of an object**. It acts as a contract that an object must follow.

**Example:**
```ts
interface Car {
  make: string;
  model: string;
  year: number;
}

const myCar: Car = { make: "Toyota", model: "Camry", year: 2022 };
```

---

### 8. Interface vs Type Alias – Which one to use?

| Feature | Interface | Type Alias |
|---|---|---|
| **Merging** | Can be merged (Declaration Merging) | Cannot be merged |
| **Extending** | Uses `extends` keyword | Uses Intersection (`&`) |
| **Types** | Only for Objects/Classes | Any type (Primitives, Unions, Tuples) |

**Rule of thumb:** Use **Interfaces** for public APIs or object shapes that might need to be extended. Use **Types** for unions, tuples, or complex logic.

---

### 9. What are Optional and Readonly properties?

-   **Optional (`?`):** Property that may or may not exist.
-   **Readonly:** Property that can only be set during initialization.

```ts
interface Book {
  readonly id: number;
  title: string;
  author?: string; // Optional
}

const b: Book = { id: 1, title: "TS Guide" };
b.id = 2; // Error: Cannot assign to 'id' because it is a read-only property
```

---

### 10. What are Tuples?

**Definition:** A **Tuple** is a specialized array with a **fixed number of elements** where each element has a known type at a specific index.

**Example:**
```ts
let person: [string, number];
person = ["Alice", 25]; // OK
person = [25, "Alice"]; // Error: Type 'number' is not assignable to 'string'
```

---

### 11. What are Enums?

**Definition:** **Enums** allow us to define a set of named constants. They make the code more readable by using descriptive names for numeric or string values.

**Example:**
```ts
enum Status {
  Pending, // 0
  Active,  // 1
  Inactive // 2
}

let current: Status = Status.Active;
console.log(current); // 1
```

---

### 12. What is Type Assertion (`as` keyword)?

**Definition:** Sometimes you know more about a value's type than TypeScript does. **Type Assertion** is a way to tell the compiler "Trust me, I know what I'm doing."

**Example:**
```ts
let someValue: unknown = "this is a string";
let strLength: number = (someValue as string).length;
```
*(Note: It doesn't actually change the data at runtime; it's just a hint for the compiler).*

---

### 13. What is the `any` type and when should you use it?

-   **What it is:** The "I don't care" type. It disables all type checking for that variable.
-   **When to use:** Only when migrating from JavaScript or working with highly dynamic data where types are impossible to predict. 
-   **Danger:** Using `any` too much defeats the purpose of using TypeScript!

---

### 14. What is the `unknown` type?

**Definition:** `unknown` is the type-safe sibling of `any`. Like `any`, it can hold any value, but unlike `any`, you **cannot perform any operations** on an `unknown` value until you verify its type (via Type Guarding).

**Example:**
```ts
let val: unknown = 10;
// val.toFixed(); // Error

if (typeof val === "number") {
  val.toFixed(); // OK!
}
```

---

### 15. What is the `never` type?

**Definition:** `never` represents values that **never occur**. It's usually the return type for functions that:
1. Always throw an error.
2. Have an infinite loop.

```ts
function throwError(msg: string): never {
  throw new Error(msg);
}
```

---

### 16. What are Literal Types?

**Definition:** You can use a specific value as a type. This is often combined with Union types.

**Example:**
```ts
type Direction = "North" | "South" | "East" | "West";
let move: Direction = "North"; // OK
move = "Up"; // Error
```

---

### 17. How do you define Function Types?

You can define the types of arguments and the return value of a function.

```ts
// Explicitly stating argument and return types
function add(a: number, b: number): number {
  return a + b;
}

// Function as a type
type MathOp = (x: number, y: number) => number;
const multiply: MathOp = (a, b) => a * b;
```

---

### 18. What are Default and Optional Parameters in functions?

```ts
function greet(name: string, greeting: string = "Hello", age?: number) {
  return `${greeting}, ${name}!`;
}

greet("Alice");           // "Hello, Alice!" (used default)
greet("Bob", "Hi");       // "Hi, Bob!"
```

---

### 19. What is an Intersection Type (`&`)?

**Definition:** An **Intersection type** combines multiple types into one. The new type will have all members of all combined types.

**Example:**
```ts
interface ErrorHandling { success: boolean; error?: string; }
interface ArtData { artists: { name: string }[]; }

type ArtResponse = ErrorHandling & ArtData;

const response: ArtResponse = {
  success: true,
  artists: [{ name: "Picasso" }]
};
```

---

### 20. What is "Type Narrowing" (Type Guards)?

**Definition:** **Type Narrowing** is the process of movement from a less specific type to a more specific type using runtime checks.

**Common ways:**
-   `typeof` (for primitives)
-   `instanceof` (for classes)
-   `in` (for property checks)
-   User-defined type guards (`parameter is Type`)

```ts
function printId(id: string | number) {
  if (typeof id === "string") {
    console.log(id.toUpperCase()); // Narrowed to string
  } else {
    console.log(id.toFixed());      // Narrowed to number
  }
}
```

---

---

## 🏗️ Section 2: Advanced Basics & Configuration

---

### 21. What are Generics in TypeScript?

**Definition:** **Generics** are like "variables for types". They allow you to create reusable components (functions, classes, or interfaces) that can work with a variety of types while still maintaining type safety.

**Example:**
```ts
function identity<T>(arg: T): T {
  return arg;
}

let result1 = identity<string>("Hello"); // T is 'string'
let result2 = identity<number>(100);     // T is 'number'
```
*Note: Using `<T>` is a convention standing for "Type".*

---

### 22. What are Access Modifiers in TypeScript Classes?

TypeScript provides three keywords to control the visibility of class members:
1.  **`public` (default):** Can be accessed from anywhere.
2.  **`private`:** Can only be accessed within the class itself.
3.  **`protected`:** Can be accessed within the class and its subclasses.

```ts
class Animal {
  private age: number;
  protected species: string;
  public name: string;
}
```

---

### 23. What is an Abstract Class?

**Definition:** An **Abstract Class** is a base class that **cannot be instantiated** directly. It is meant to be extended by other classes. It can contain **Abstract Methods** (methods that have no implementation and MUST be implemented by subclasses).

```ts
abstract class Shape {
  abstract getArea(): number; // Subclasses must implement this
}

class Circle extends Shape {
  constructor(public radius: number) { super(); }
  getArea() { return Math.PI * this.radius ** 2; }
}
```

---

### 24. What is the `readonly` modifier in Classes?

In a class, `readonly` means the property can only be assigned during declaration or inside the constructor.

```ts
class Octopus {
  readonly name: string;
  readonly numberOfLegs: number = 8;
  constructor(theName: string) {
    this.name = theName;
  }
}
```

---

### 25. What is Parameter Property shorthand in Classes?

TypeScript allows you to declare and initialize a class property in a single line within the constructor.

```ts
class User {
  // Shorthand for:
  // private id: number; constructor(id: number) { this.id = id; }
  constructor(private id: number, public name: string) {}
}
```

---

### 26. What is a Type Predicate (`is` keyword)?

**Definition:** A **Type Predicate** is a way to create a custom type guard function. It tells TypeScript exactly what type a variable is if the function returns `true`.

```ts
function isString(val: any): val is string {
  return typeof val === "string";
}

let x: unknown = "hello";
if (isString(x)) {
  console.log(x.toUpperCase()); // x is now treated as a string
}
```

---

### 27. What are Index Signatures?

**Definition:** Index signatures are used when you don't know all the names of an object's properties ahead of time, but you do know the **type of values** they will hold.

```ts
interface SalaryMap {
  [name: string]: number; // Any key (string) maps to a value (number)
}

const salaries: SalaryMap = {
  Alice: 5000,
  Bob: 6000
};
```

---

### 28. What is the `keyof` operator?

**Definition:** The **`keyof`** operator takes an object type and produces a string or numeric literal union of its keys.

```ts
interface User { id: number; name: string; }
type UserKeys = keyof User; // "id" | "name"
```

---

### 29. What is the `typeof` operator in TypeScript?

In TypeScript, **`typeof`** can be used in a **type context** to refer to the type of a variable or property.

```ts
let point = { x: 10, y: 20 };
type Point = typeof point; // { x: number, y: number }
```

---

### 30. What is `tsconfig.json`?

**Definition:** **`tsconfig.json`** is a configuration file that tells the TypeScript compiler (`tsc`) how to compile your code.

**Common options:**
-   `target`: The version of JavaScript to output (e.g., `ES6`).
-   `module`: The module system to use (e.g., `CommonJS`, `ESNext`).
-   `rootDir`: Where your TS source files are.
-   `outDir`: Where compiled JS files should go.
-   `strict`: Enables all strict type-checking options (highly recommended!).
-   `baseUrl`: Base directory to resolve non-relative module names.

---

### 31. What are Declaration Files (`.d.ts`)?

**Definition:** Declaration files are files that **only contain type information** (no actual code). They are used to provide types for existing JavaScript libraries.

**Example:** If you install `lodash`, you also install `@types/lodash` which contains the `.d.ts` files describing lodash functions.

---

### 32. What is the `@types` scope in npm?

**Definition:** This is where you find **DefinitelyTyped** definitions for libraries that weren't written in TypeScript. Most popular JS libraries have a corresponding `@types` package.

---

### 33. How does TypeScript handle Modules?

TypeScript uses the same syntax as ES6 for modules: `import` and `export`.
-   **Internal Modules:** (Old name for namespaces).
-   **External Modules:** (Standard JS modules).

---

### 34. What are Utility Types (Part 1)?

TypeScript provides several built-in utility types to transform existing types:
1.  **`Partial<T>`**: Makes all properties optional.
2.  **`Required<T>`**: Makes all properties required.
3.  **`Readonly<T>`**: Makes all properties read-only.
4.  **`Pick<T, Keys>`**: Creates a type by picking specific keys from `T`.
5.  **`Omit<T, Keys>`**: Creates a type by removing specific keys from `T`.

```ts
interface User { id: number; name: string; email: string; }
type UserPreview = Pick<User, "id" | "name">; // { id: number; name: string }
```

---

### 35. TypeScript Best Practices for Beginners?

1.  **Enable `strict` mode:** Always do this in your `tsconfig.json`.
2.  **Avoid `any`:** Use `unknown` or specific interfaces instead.
3.  **Use Type Inference:** Don't explicitly type every single variable if TS can guess it correctly.
4.  **Interfaces for Objects:** Prefer `interface` for object shapes that describe "public" structures.
5.  **Explicit Return Types:** For complex functions, explicitly define the return type for better readability and to catch bugs earlier.
6.  **Organize with Modules:** Keep your code modular and use clear `import/export`.

---

*End of TypeScript Basics Reference*
