# React Basics – Complete Beginner Interview Reference

---

## ⚛️ Section 1: Core Fundamentals

---

### 1. What is React?

**Definition:** **React** (also known as React.js or ReactJS) is an **open-source, front-end JavaScript library** developed by Facebook for building user interfaces, specifically for **single-page applications (SPAs)**. It's used for handling the **view layer** in web and mobile apps.

**Key Features:**
-   **Declarative:** You describe *what* you want the UI to look like for various states, and React manages the *how* (updating the DOM).
-   **Component-Based:** The UI is broken down into small, isolated, and reusable pieces called components.
-   **Learn Once, Write Anywhere:** You can use React to build web apps (React), mobile apps (React Native), and even VR apps.
-   **Virtual DOM:** Uses a light-weight representation of the real DOM to improve performance.

**Why use it?**
-   **Fast:** The Virtual DOM makes UI updates highly efficient.
-   **Modular:** Components make code easier to maintain and reuse.
-   **Strong Ecosystem:** Huge community support, libraries (Redux, React Router), and developer tools.

---

### 2. What is JSX?

**Definition:** **JSX** stands for **JavaScript XML**. It is a syntax extension for JavaScript that allows you to write HTML-like code directly inside your JavaScript files. It makes the code more readable and expressive when defining UI structures.

**How it works:**
Browsers cannot read JSX directly. It must be **compiled/transpiled** into regular JavaScript `React.createElement()` calls using tools like **Babel**.

**Example:**
```jsx
// JSX Code
const element = <h1 className="title">Hello, World!</h1>;

// Transpiled Code (what the browser actually runs)
const element = React.createElement(
  'h1',
  { className: 'title' },
  'Hello, World!'
);
```

**Key rules of JSX:**
1.  **Return a single element:** You must wrap multiple elements in a fragment `<>...</>` or a `<div>`.
2.  **Close all tags:** Even self-closing tags like `<img />` or `<br />` must be closed.
3.  **CamelCase for attributes:** Use `className` instead of `class`, `htmlFor` instead of `for`, and `onClick` instead of `onclick`.
4.  **Embed JS expressions:** Use curly braces `{}` to include JavaScript logic inside JSX.

---

### 3. What is the Virtual DOM?

**Definition:** The **Virtual DOM (VDOM)** is a lightweight, in-memory **representation** of the real browser DOM. It’s a plain JavaScript object that mirrors the structure of the actual DOM.

**How it solves performance issues:**
Updating the real DOM is expensive (slow) because it triggers layout recalibration and repainting. React minimizes these updates with a process called **Reconciliation**.

**The Process:**
1.  **Render:** When state changes, React creates a *new* Virtual DOM tree.
2.  **Diffing:** React compares (diffs) the new VDOM with the *previous* VDOM tree.
3.  **Patching:** React identifies exactly which parts changed and updates **only those specific parts** in the real DOM.

**Analogy:**
If the real DOM is your entire house, the Virtual DOM is a **blueprint** of your house. Instead of rebuilding the whole house every time you want to paint a wall, you update the blueprint, find the difference, and paint only that specific wall.

---

### 4. What are Components?

**Definition:** **Components** are the building blocks of a React application. They are independent, reusable pieces of UI that can be composed together to build complex interfaces. A component is essentially a JavaScript function (or class) that returns JSX.

**Types of Components:**

#### 1. Functional Components (Modern standard):
Simpler, easier to test, and use **Hooks** for state and lifecycle.
```jsx
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
```

#### 2. Class Components (Legacy):
Older way of writing components (pre-React 16.8). They use ES6 classes and have built-in state and lifecycle methods.
```jsx
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

**Key Principles:**
-   **Composition:** Components can contain other components (Parent/Child).
-   **Reusability:** Write once, use many times with different data (Props).
-   **Independence:** Each component manages its own logic and UI.

---

### 5. What are Props?

**Definition:** **Props** (short for "properties") are **read-only** inputs passed from a parent component to a child component. They are used to pass data and configuration down the component tree.

**Key Characteristics:**
-   **Immutable:** A component should never modify its own props.
-   **Uni-directional:** Data flows one way — from parent to child.
-   **Functional Arguments:** In functional components, they are passed as the first argument.

**Example:**
```jsx
// Parent Component
function App() {
  return <User Card name="Alice" age={25} />;
}

// Child Component
function UserCard(props) {
  return (
    <div>
      <h3>Name: {props.name}</h3>
      <p>Age: {props.age}</p>
    </div>
  );
}
```

---

### 6. What is State?

**Definition:** **State** is a built-in object that allows a component to **manage its own data**. Unlike props, state is **private** and fully controlled by the component itself. When state changes, React automatically re-renders the component to reflect the new data.

**Difference from Props:**
| Feature | Props | State |
|---|---|---|
| **Source** | Passed from Parent | Defined within Component |
| **Mutability** | Immutable (Read-only) | Mutable (Changed via updater function) |
| **Ownership** | External to component | Internal to component |

**Example (Functional):**
```jsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0); // State initialization

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  );
}
```

---

### 7. What is the `useState` Hook?

**Definition:** `useState` is a **Hook** that lets you add React state to functional components. It is the modern replacement for `this.state` in class components.

**Syntax:**
```js
const [state, setState] = useState(initialValue);
```
1.  **`state`:** The current state value.
2.  **`setState`:** A function to update the state.
3.  **`initialValue`:** The value the state starts with (can be a string, number, array, or object).

**Important rules:**
-   **Don't mutate directly:** Never do `state = newValue`. Always use `setState(newValue)`.
-   **Asynchronous:** State updates are scheduled, not immediate.
-   **Functional Updates:** If the new state depends on the previous state, use the functional form:
    ```js
    setCount(prevCount => prevCount + 1);
    ```

---

### 8. What is the `useEffect` Hook?

**Definition:** `useEffect` lets you perform **side effects** in functional components. Side effects are actions that happen "outside" the normal render flow, such as fetching data, manually changing the DOM, or setting up subscriptions.

**Syntax:**
```js
useEffect(() => {
  // Effect logic here...

  return () => {
    // Optional Cleanup logic here (e.g., clear timer)
  };
}, [dependencies]);
```

**The Dependency Array `[]`:**
-   **No array:** Runs on **every** render.
-   **Empty array `[]`:** Runs only **once** (after the initial mount). Like `componentDidMount`.
-   **With values `[prop, state]`:** Runs whenever **any value** in the array changes.

**Example (Data Fetching):**
```jsx
useEffect(() => {
  fetch('/api/data')
    .then(res => res.json())
    .then(data => setData(data));
}, []); // Runs once on mount
```

---

### 9. What is Conditional Rendering?

**Definition:** **Conditional Rendering** is the ability to show different UI elements based on certain conditions (like in regular JavaScript `if` or `ternary` statements).

**Ways to implement:**

#### 1. Ternary Operator (`? :`):
Great for "either-or" scenarios.
```jsx
{isLoggedIn ? <UserDashboard /> : <LoginForm />}
```

#### 2. Logical AND (`&&`):
Great for "show or hide" scenarios.
```jsx
{hasNotifications && <Badge count={notifications.length} />}
```

#### 3. If/Else (Outside JSX):
```jsx
function Greeting({ user }) {
  if (user) {
    return <h1>Welcome back, {user.name}!</h1>;
  }
  return <h1>Please sign in.</h1>;
}
```

---

### 10. What are Keys in React Lists?

**Definition:** **Keys** are unique identifiers used by React to keep track of which items in a list have changed, been added, or been removed. They are essential for performance during the **Reconciliation** process.

**Why are they needed?**
Without keys, if you reorder a list, React might re-render every item because it can't tell which one moved where. With keys, React can efficiently reorder the DOM nodes without destroying and recreating them.

**Example:**
```jsx
const todoItems = todos.map((todo) =>
  <li key={todo.id}>
    {todo.text}
  </li>
);
```

**Rules for Keys:**
-   **Stable and Unique:** Use IDs from your database (like `todo.id`). Avoid using random numbers like `Math.random()`.
-   **Don't use Array Index (mostly):** Avoid the array index `(item, index) => ...` if the list can be reordered, filtered, or items can be added in the middle, as it can cause UI bugs.

---
## 🎣 Section 2: Props, State & Hooks (Advanced Basics)

---

### 11. What is Prop Drilling?

**Definition:** **Prop Drilling** is the process of passing data through multiple layers of components (from Parent to Grandchild to Great-Grandchild) even if the middle components don't need the data themselves.

**The Problem:**
-   **Verbose:** You have to pass the same props through many components.
-   **Maintenance:** If you change the prop name, you have to update it in every component along the path.
-   **Coupling:** Components become tightly coupled to data they don't even use.

**The Solution:**
To avoid prop drilling, you can use:
1.  **Context API:** Built-in React feature for global state.
2.  **State Management Libraries:** Redux, ZuStand, Recoil.
3.  **Component Composition:** Passing components as props instead of raw data.

---

### 12. What is the Context API?

**Definition:** The **Context API** is a built-in React feature that allows you to share data (like themes, user info, or language settings) globally across the entire component tree without manually passing props at every level.

**How to use it:**
1.  **Create Context:** `const ThemeContext = React.createContext('light');`
2.  **Provide Context:** Wrap the parent component with `<ThemeContext.Provider value="dark">`.
3.  **Consume Context:** Use the `useContext` hook in any child component.

**Example:**
```jsx
const ThemeContext = createContext('light');

function App() {
  return (
    <ThemeContext.Provider value="dark">
      <Toolbar />
    </ThemeContext.Provider>
  );
}

function Toolbar() {
  return <ThemedButton />; // No props passed here!
}

function ThemedButton() {
  const theme = useContext(ThemeContext); // Directly access "dark"
  return <button className={theme}>I am {theme} themed!</button>;
}
```

---

### 13. What is "Lifting State Up"?

**Definition:** **Lifting State Up** is a pattern where you move the state from child components to their **closest common ancestor**. This is necessary when multiple sibling components need to share and stay in sync with the same data.

**Example:**
If you have two inputs for Celsius and Fahrenheit, and you want them to stay in sync, you move the `temperature` state to their parent component and pass down the value and change handler as props.

```jsx
// Parent
function Calculator() {
  const [temp, setTemp] = useState('');
  
  return (
    <>
      <Input value={temp} onChange={setTemp} label="Celsius" />
      <Input value={temp} onChange={setTemp} label="Fahrenheit" />
    </>
  );
}
```

---

### 14. What are Controlled vs Uncontrolled Components?

**Controlled Components:**
The component's state is **managed by React**. The input value is tied to a React state variable, and changes are handled by an `onChange` function.
-   **PROS:** Validation on the fly, immediate feedback, easy to reset.
-   **CONS:** Slightly more boilerplate.

**Uncontrolled Components:**
The component's state is **managed by the DOM**. You use a **`ref`** (via `useRef`) to pull the value from the DOM only when needed (e.g., on form submission).
-   **PROS:** Good for integrating with non-React libraries or simple forms.
-   **CONS:** Harder to validate instantly.

```jsx
// Controlled
const [val, setVal] = useState('');
<input value={val} onChange={(e) => setVal(e.target.value)} />

// Uncontrolled
const inputRef = useRef();
<input ref={inputRef} />
console.log(inputRef.current.value);
```

---

### 15. What is the `useRef` Hook?

**Definition:** `useRef` returns a mutable ref object whose `.current` property is persistent across re-renders. It is primarily used for two things:
1.  **Accessing DOM elements:** To focus an input, scroll to a position, or integrate with 3rd-party DOM libraries.
2.  **Storing mutable values:** To store a value that *doesn't* trigger a re-render when it changes (like timers, previous state values).

**Example (DOM Access):**
```jsx
function TextInputWithFocus() {
  const inputEl = useRef(null);
  const onButtonClick = () => {
    inputEl.current.focus(); // Directly access the DOM node
  };
  return (
    <>
      <input ref={inputEl} type="text" />
      <button onClick={onButtonClick}>Focus the input</button>
    </>
  );
}
```

---

### 16. What is `React.memo()`?

**Definition:** `React.memo` is a **Higher-Order Component (HOC)** used for performance optimization. It "memoizes" (caches) a functional component, meaning React will **skip re-rendering** it if its props haven't changed.

**When to use:**
-   When a component renders often with the same props.
-   When the component is pure and its render result only depends on props.

**Example:**
```jsx
const MyComponent = React.memo(function MyComponent(props) {
  /* render using props */
});
```

---

### 17. What is `useMemo` vs `useCallback`?

Both are Hooks for optimization by caching values/functions.

| Hook | Purpose | Returns |
|---|---|---|
| **`useMemo`** | Memoizes the **result of a calculation**. | A memoized **value**. |
| **`useCallback`**| Memoizes a **function definition**. | A memoized **function**. |

**`useMemo` example:**
```js
const expensiveValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

**`useCallback` example:**
```js
const memoizedCallback = useCallback(() => {
  doSomething(a, b);
}, [a, b]);
```
(Useful when passing functions to memoized child components to prevent unnecessary re-renders).

---

### 18. What are Custom Hooks?

**Definition:** A **Custom Hook** is a JavaScript function whose name starts with **"use"** and that can call other Hooks. It’s a mechanism to reuse **stateful logic** between different components without repeating code.

**Example (window width tracker):**
```jsx
function useWindowWidth() {
  const [width, setWidth] = useState(window.innerWidth);
  
  useEffect(() => {
    const handleResize = () => setWidth(window.innerWidth);
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return width;
}
```

---

### 19. What is a "Fragment"?

**Definition:** A **Fragment** is a React component that lets you group multiple children elements together **without adding an extra node (like a `<div>`) to the DOM**.

**Syntax:**
```jsx
// Long form
<React.Fragment>
  <li>Item 1</li>
  <li>Item 2</li>
</React.Fragment>

// Short form (cannot take keys)
<>
  <li>Item 1</li>
  <li>Item 2</li>
</>
```

---

### 20. What are the Rules of Hooks?

To ensure hooks work correctly, you must follow two main rules:
1.  **Only Call Hooks at the Top Level:** Don’t call Hooks inside loops, conditions, or nested functions. This ensures they are called in the same order every time a component renders.
2.  **Only Call Hooks from React Functions:** Call them from React functional components or custom Hooks. Don’t call them from regular JavaScript functions.

---

## 🔄 Section 3: Component Lifecycle & Rendering

---

### 21. What is the Component Lifecycle?

**Definition:** The **Component Lifecycle** refers to the series of phases a React component goes through, starting from its creation to its destruction. 

**Phases:**
1.  **Mounting:** When the component is being created and inserted into the DOM (e.g., `componentDidMount` in classes, or `useEffect` with `[]` in functional).
2.  **Updating:** When the component’s props or state change, causing a re-render (e.g., `componentDidUpdate` in classes, or `useEffect` with dependencies).
3.  **Unmounting:** When the component is being removed from the DOM (e.g., `componentWillUnmount` in classes, or the **cleanup** function in `useEffect`).

---

### 22. What are Side Effects in React?

**Definition:** **Side Effects** are any operations that affect something outside the scope of the function being executed. In React, this includes:
-   Fetching data from an API.
-   Manually changing the DOM.
-   Subscriptions (like WebSockets or timers).
-   Setting up event listeners.

**Where to handle them:** Side effects should be handled in **`useEffect`** to ensure they don't block the UI rendering.

---

### 23. What is React Reconciliation?

**Definition:** **Reconciliation** is the internal algorithm React uses to compare the updated Virtual DOM with the previous one to determine which parts of the real DOM need to be updated. It balances **speed** and **efficiency**.

---

### 24. What is a "Single Page Application" (SPA)?

**Definition:** An **SPA** is a web application that loads a single HTML page and dynamically updates that page as the user interacts with the app. Instead of loading new pages from the server, React (or a router) dynamically re-renders parts of the current page.

**Benefits:**
-   **Faster Navigation:** No full page reloads.
-   **Smoother UX:** Transitions feel like a mobile app.
-   **Reduced Server Load:** The server only sends data (JSON) instead of full HTML pages.

---

### 25. What is the purpose of "Strict Mode" in React?

**Definition:** **`<React.StrictMode>`** is a tool for highlighting potential problems in an application. It doesn't render any visible UI but it:
-   Identifies components with unsafe lifecycles.
-   Warns about legacy API usage.
-   Detects unexpected side effects by **double-invoking** components in development.

---

## 📋 Section 4: Forms, Lists & Navigation

---

### 26. How do you handle Forms in React?

To handle forms, you typically use **Controlled Components**, where the form data is handled by the component's state.

**Example:**
```jsx
function MyForm() {
  const [name, setName] = useState("");

  const handleSubmit = (event) => {
    event.preventDefault();
    alert("A name was submitted: " + name);
  }

  return (
    <form onSubmit={handleSubmit}>
      <input 
        type="text" 
        value={name} 
        onChange={(e) => setName(e.target.value)} 
      />
      <button type="submit">Submit</button>
    </form>
  );
}
```

---

### 27. What is React Router?

**Definition:** **React Router** is a standard library for routing in React. It enables navigation among different components in an SPA, allows changing the browser URL, and keeps the UI in sync with the URL.

**Main Components:**
1.  **`<BrowserRouter>`:** Wraps the entire app to enable routing.
2.  **`<Routes>`:** A container for individual route definitions.
3.  **`<Route>`:** Defines a path and the component that should render for that path.
4.  **`<Link>`:** Used to navigate between routes without reloading the page.

---

### 28. What is an Event in React?

**Definition:** Events in React are similar to handling events on DOM elements, with some syntax differences:
-   React events are named using **camelCase** (e.g., `onClick`, `onChange`).
-   In JSX, you pass a **function** as the event handler rather than a string.

**Example:**
```jsx
<button onClick={handleClick}>Click Me</button>
```

---

### 29. What is an "Error Boundary"?

**Definition:** **Error Boundaries** are React components that catch JavaScript errors anywhere in their child component tree, log those errors, and display a fallback UI instead of the component tree that crashed.

*(Note: Currently, Error Boundaries can only be created using **class components**).*

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  render() {
    if (this.state.hasError) {
      return <h1>Something went wrong.</h1>;
    }
    return this.props.children;
  }
}
```

---

### 30. What is a Higher-Order Component (HOC)?

**Definition:** An **HOC** is a pattern where a function takes a component and returns a *new* component with added functionality. It is used for **logic reuse**.

**Example:**
```javascript
const withLogging = (WrappedComponent) => {
  return function(props) {
    console.log("Rendering component...");
    return <WrappedComponent {...props} />;
  }
}
```

---

### 31. How to style a React component?

There are several ways:
1.  **CSS Stylesheets:** Regular external `.css` files.
2.  **Inline Styling:** Using a JavaScript object with camelCase properties:
    ```jsx
    <div style={{ color: "blue", fontSize: "20px" }}></div>
    ```
3.  **CSS Modules:** Locally scoped CSS (e.g., `Button.module.css`).
4.  **Styled Components:** Logic and styles combined in JS (using `styled-components` library).
5.  **Tailwind CSS:** Utility-first CSS framework.

---

### 32. What is the difference between an Element and a Component?

| Feature | React Element | React Component |
|---|---|---|
| **What it is** | A plain object describing a DOM node. | A function or class that returns elements. |
| **Example** | `const el = <h1>Hi</h1>;` | `function MyComp() { ... }` |
| **Analogy** | A photo of a dish. | The chef and the recipe. |

---

### 33. What is Webpack / Babel (in React context)?

-   **Babel:** A compiler that turns JSX and ES6+ code into plain JavaScript that older browsers can understand.
-   **Webpack:** A module bundler that takes all your JS, CSS, and image files and bundles them into a single file (or a few files) for the browser to load efficiently.

---

### 34. What is "React Fiber"?

**Definition:** **Fiber** is the internal engine of React (introduced in React 16) that handles the **Reconciliation** process. Its main goal is to increase its suitability for areas like animation, layout, and gestures. It allows React to pause, resume, or abort work as needed (Concurrency).

---

### 35. What is the default export vs named export?

-   **Default Export:** One per module. `export default MyComponent;` (Import: `import MyComponent from './file'`).
-   **Named Export:** Multiple per module. `export const MyComp = () => ...` (Import: `import { MyComp } from './file'`).

---

### 40. Common React Best Practices Checklist:

```
✅ Use Functional Components and Hooks.
✅ Keep components small and focused (Single Responsibility).
✅ Use descriptive prop and state names.
✅ Use unique and stable keys for lists.
✅ Avoid prop drilling (use Context for global state).
✅ Always clean up side effects in useEffect (timers, event listeners).
✅ Use fragments <> to avoid extra DOM nodes.
✅ Destructure props for cleaner code.
✅ Put common logic in Custom Hooks.
✅ Handle loading and error states in UI.
```

---

*End of React Basics Reference*

---

### 41. What is the difference between Shadow DOM and Virtual DOM?

-   **Shadow DOM:** A browser-native technology used to encapsulate styles and markup in **Web Components**. It keeps the component's internal DOM separate from the main document.
-   **Virtual DOM:** A React-specific concept (a JavaScript object) used to improve performance by minimizing direct manipulation of the real DOM through a "diffing" process.

---

### 42. What are "Portals" in React?

**Definition:** **Portals** provide a way to render children into a DOM node that exists **outside the DOM hierarchy** of the parent component.

**Common use case:** Modals, tooltips, or floating menus that need to break out of a container with `overflow: hidden` or `z-index` issues.

```jsx
ReactDOM.createPortal(child, container)
```

---

### 43. What is `React.Children.map`?

**Definition:** It is a professional helper utility for dealing with the `this.props.children` opaque data structure. It ensures that you can iterate over children even if it's a single element, an array, or null.

```jsx
React.Children.map(this.props.children, child => {
  return <li>{child}</li>;
});
```

---

### 44. What is the purpose of `forwardRef`?

**Definition:** **Ref forwarding** is a technique for automatically passing a **ref** through a component to one of its children. This is useful for reusable component libraries where you want the parent to be able to focus an input inside a custom `MyInput` component.

```jsx
const MyInput = React.forwardRef((props, ref) => (
  <input ref={ref} className="fancy-input" />
));
```

---

### 45. What is `React.lazy` and `Suspense`?

**Definition:** **`React.lazy`** lets you render a dynamic import as a regular component. It helps in **code-splitting**, reducing the initial bundle size. **`Suspense`** allows you to show a fallback (like a spinner) while the lazy component is loading.

```jsx
const OtherComponent = React.lazy(() => import('./OtherComponent'));

function MyComponent() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <OtherComponent />
    </Suspense>
  );
}
```

---

### 46. What is a "SyntheticEvent" in React?

**Definition:** React wraps the browser’s native events into a **SyntheticEvent** object to ensure cross-browser consistency (it works exactly the same in Chrome, Safari, and Firefox). It has the same interface as native events (`stopPropagation()`, `preventDefault()`), but it’s managed by React.

---

### 47. What is "Hydration" in React?

**Definition:** **Hydration** is the process where React attaches event listeners to the static HTML that was sent from the server (SSR). It "wakes up" the static HTML and makes it interactive by turning it into a full React application.

---

### 48. What is the difference between `useMemo` and `React.memo`?

-   **`React.memo`:** An HOC that prevents a **component** from re-rendering if its props haven't changed.
-   **`useMemo`:** A Hook that prevents an **expensive calculation** from running again if its dependencies haven't changed.

---

### 49. What is `flushSync`?

**Definition:** `flushSync` is a rare utility that forces React to **synchronously** flush any updates inside the provided callback. This ensures that the DOM is updated immediately before the next line of code runs. (Use sparingly!)

---

### 50. What is a "Strict Mode" (Recap for Interviews)?

**Strict Mode** is a development-only tool that checks for potential problems:
-   Identifies discovery of unsafe lifecycles.
-   Warning about legacy string ref API usage.
-   Warning about deprecated findDOMNode usage.
-   Checking for unexpected side effects by **double-rendering** components.

---

*End of React Basics Reference*
