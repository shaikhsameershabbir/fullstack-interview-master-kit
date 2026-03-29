# Module 3: Frontend Architecture (React Context & Next.js)

This module explores the inner workings of React. Senior Frontend/Full-Stack engineers must understand identical UI results can have vastly different CPU and rendering profiles.

---

## 6️⃣ React Internals & Performance

### 18. What is React Reconciliation and Fiber Architecture?
**Deep Dive**: 
- **Reconciliation**: The diffing algorithm. When state changes, React builds a new Virtual DOM tree. It compares the new tree with the old tree (using heuristics like `keys` on arrays and `Component Type` checks) to figure out the minimal set of raw DOM mutations required (e.g., `document.getElementById('x').className = "new"`).
- **Fiber**: React 16 completely rewrote the reconciler. The old "Stack" reconciler was synchronous; once an update started, it locked the browser's main thread until finished, causing input lag.
- **The Solution**: Fiber breaks the update process into "units of work" (Fibers). React can pause rendering a heavy unmounted dashboard, yield control back to the browser so the user can type in an input box smoothly, and then resume the dashboard rendering where it left off.

### 19. How does the Virtual DOM actually work? Does it make React "fast"?
**Deep Dive**: 
- **The Myth**: The Virtual DOM does *not* make React computationally faster than vanilla JS writing to the real DOM (`innerHTML`). Creating JS objects in memory is fast, but it still adds overhead.
- **The Reality**: The VDOM makes writing *declarative* code fast enough to be scalable. Without it, developers would manually write thousands of `if (prevName !== newName) { updateDOM() }` statements for every pixel on the screen. The VDOM calculates the optimal batch of DOM writes automatically.
- **Current Paradigms**: Frameworks like **Svelte** and **SolidJS** proved the VDOM is an unnecessary memory overhead. They use AOT (Ahead of Time) compilation or fine-grained reactivity (Signals) to surgically update DOM nodes without ever building a Virtual Tree.

### 20. What are unnecessary re-renders, and how do you stop them?
**Deep Dive**: 
- *The Cause*: In React, if a parent component's state changes, *every single child component re-renders by default*, even if their props didn't change.
- **React.memo**: Wraps a component. It performs a shallow comparison (`===`) of previous props and new props. If they are identical, React skips rendering the child.
- **useMemo / useCallback**: 
    - A parent component defines `const config = { theme: 'dark' }`. Every time the parent renders, an entirely new object reference is created in memory.
    - If the parent passes `config` to a child wrapped in `React.memo`, the memoization fails immediately because `prevConfig !== newConfig` (different memory addresses).
    - Wrapping it in `const config = useMemo(() => ({ theme: 'dark' }), [])` preserves the exact memory address across renders, allowing `React.memo` to work properly.

### 21. When should you explicitly NOT use useEffect?
**Deep Dive**: 
- `useEffect` is widely abused as a way to "react" to state changes (Derived State).
- **Anti-Pattern (Derived State)**:
  ```javascript
  const [firstName, setFirstName] = useState('');
  const [fullName, setFullName] = useState('');
  // DO NOT DO THIS:
  useEffect(() => { setFullName(firstName + ' Smith'); }, [firstName]);
  ```
- *Why it's bad*: It forces React to render the component *twice* (once with just the first name, and immediately a second time after the effect fires to set the full name).
- *Solution*: Calculate derived state functionally during the render cycle: `const fullName = firstName + ' Smith';`.
- **Anti-Pattern (Fetching Data)**: Using `useEffect` to fetch data leads to race conditions, zero caching, and "waterfalls." Senior engineers use libraries like `React Query` (TanStack Query) or `SWR`, which handle cache-invalidation, deduplication, and retries natively.

---

## 7️⃣ Next.js Server & Edge Paradigms

### 22. SSR vs CSR vs SSG vs ISR
**Deep Dive**: 
- **CSR (Client-Side Rendering) - Stand React App**: Sends an empty `index.html` and a 2MB Javascript bundle. The browser downloads the JS and executes it to build the HTML. *Terrible* for SEO and Time-To-Interactive on slow phones.
- **SSR (Server-Side Rendering)**: A Node.js server generates the final HTML string dynamically for *every* incoming request. Great for SEO, but heavy CPU cost on the server.
- **SSG (Static Site Generation)**: The HTML string is generated *once* during the CI/CD build process (`next build`). Highly performant, easily cached on CDNs. Terrible if data changes every 5 seconds.
- **ISR (Incremental Static Regeneration)**: Best of both worlds. A page is statically generated, but given a `revalidate: 60` tag. After 60 seconds, the *next* request triggers a background recreation of the HTML. The user gets the stale page instantly, but the CDN caches the fresh page for the next user.

### 23. What are React Server Components (RSC - App Router)?
**Deep Dive**: 
- Traditionally, even in SSR, all React components were eventually shipped to the browser as a JavaScript bundle to handle interactivity ("hydration").
- **Server Components** execute *exclusively* on the server and are never included in the browser's JS bundle.
- *Benefits*: You can import the massive `moment.js` library or run a direct PostgreSQL query `SELECT * FROM users` directly inside the React component. That heavy node library is executed server-side, and the component outputs pure serialized HTML to the client. This drastically reduces the browser's Javascript footprint.
- *Constraints*: Server components cannot use `useState`, `useEffect`, or `onClick` (because they don't exist in the browser). If a component needs interactivity, it must be explicitly marked with the `"use client"` directive.

### 24. What is the Edge Runtime vs Node.js Runtime in Next.js?
**Deep Dive**: 
- **Node.js Runtime**: Full access to the filesystem (`fs`), child processes, and native C++ bindings (like `bcrypt`). But spinning up a Node.js Lambda function takes 500ms (Cold Start).
- **Edge Runtime**: A highly constrained V8 environment executed directly on global CDN nodes (like Cloudflare Workers). It cannot use `fs` or native Node APIs.
- *Benefit*: An Edge function boots up in 0 milliseconds and executes geographically closest to the user. Perfect for ultra-fast Next.js Middleware handling A/B testing flag retrieval or IP-based Rate Limiting before the request ever reaches your actual backend servers.
