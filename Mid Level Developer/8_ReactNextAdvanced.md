# React & Next.js Advanced – Mid-Level Developer Interview Reference

---

## ⚛️ Section 1: React Core Architecture & Rendering

---

### 1. What is the React Fiber Architecture?

**Definition:** Fiber is the rewrite of the React reconciliation engine. It allows React to **break down rendering work** into small units and spread them across multiple frames.

**Internal Mechanics:**
1.  **Work in Chunks:** Before Fiber, rendering was "Stack-based" and couldn't be interrupted. Fiber is "Pointer-based" (a linked tree of fibers).
2.  **Concurrency:** React can pause a low-priority render (e.g., an off-screen list) to handle a high-priority user input (e.g., typing in a search bar).
3.  **Two-Phase Rendering:**
    -   **Render Phase (Asynchronous):** React calculates changes (diffing). This can be paused and restarted.
    -   **Commit Phase (Synchronous):** React applies changes to the DOM. This must be finished in one go to avoid inconsistent UI.

---

### 2. Concurrent Mode: `useTransition` and `useDeferredValue`

**`useTransition`:** Used to mark a state update as a "Transition" (non-urgent).
-   *Use Case:* Switching tabs. If the new tab takes 500ms to render, React will keep the old tab visible and showing a "Loading" state instead of freezing the whole UI.

**`useDeferredValue`:** Used to "defer" updating a specific value.
-   *Use Case:* Search Bar. The input is urgent (must reflect keystrokes immediately), but the results list is not. You defer the `searchQuery` for the results list so it doesn't slow down the typing experience.

---

### 3. Server Components (RSC) vs Client Components

**Server Components (Default in Next.js App Router):**
-   **Where:** Rendered **only on the server**.
-   **Payload:** Zero JavaScript sent to the client. This dramatically reduces bundle size.
-   **Capability:** Can fetch data directly from the Database (no need for `/api` routes).
-   **Limitation:** No `useState`, `useEffect`, or Browser APIs (window).

**Client Components (`'use client'`):**
-   **Where:** Rendered on the server (initial) and hydrated on the client.
-   **Payload:** Shared JS bundle sent to the client.
-   **Capability:** Interactive elements, event listeners, state, and browser APIs.

---

### 4. What is "Hydration" and why do "Hydration Mismatches" occur?

**Definition:** Hydration is the process of attaching event listeners to the static HTML that was initially rendered by the server.

**Hydration Mismatch:**
-   **Problem:** The HTML generated on the server is different from what the client tries to render (e.g., using `new Date()` or `window.innerWidth` during initial render).
-   **Impact:** React throws a warning and might have to re-render the entire branch, losing all performance benefits of SSR.
-   **Fix:** Use `useEffect` to trigger client-only logic after the first mount, or use the `suppressHydrationWarning` prop (as a last resort).

---

### 5. Advanced Pattern: Compound Components

**Definition:** A pattern where a parent component manages the state and shares it with its children (usually via Context), providing a highly flexible and readable API.

**Example:**
```tsx
<Tabs defaultValue="home">
  <Tabs.List>
    <Tabs.Trigger value="home">Home</Tabs.Trigger>
    <Tabs.Trigger value="settings">Settings</Tabs.Trigger>
  </Tabs.List>
  <Tabs.Content value="home">Welcome Home</Tabs.Content>
</Tabs>
```
**Benefit:** Zero "Prop Drilling". The user of the component has full control over the layout without the component library having to provide 50 different props for every possible variation.

---

### 6. React Performance: `Selective Hydration` & `Streaming`

**Streaming SSR:**
Next.js allows you to send the page "piece by piece" using `<Suspense>`.
1.  The layout and fixed content are sent immediately.
2.  The slow component (e.g., `ReviewsList`) is sent as a "Loading" skeleton.
3.  Once the data is ready, the server "streams" the HTML and JS for that component to the client, which React then swaps in.

**Selective Hydration:**
React doesn't wait for the whole page to hydrate. It hydrates the components that the user **interacts with first** (e.g., if the user clicks a button, React prioritizes hydrating that button's region).

---

### 7. Next.js Data Fetching & Caching (App Router)

-   **`fetch(url, { cache: 'force-cache' })`**: Static Data Fetching (like `getStaticProps`).
-   **`fetch(url, { cache: 'no-store' })`**: Dynamic Data Fetching (like `getServerSideProps`).
-   **`fetch(url, { next: { revalidate: 3600 } })`**: Incremental Static Regeneration (ISR).

**Request Deduping:** Next.js automatically deduplicates `fetch` calls. If you call `fetch('/api/user')` in 5 different components at once, only ONE network request is actually made.

---

### 8. Server Actions: Form Handling without API Routes

**Definition:** A way to call a server-side function directly from a client component (e.g., a form `action`).

**Benefits:**
1.  **Progressive Enhancement:** The form works even if JavaScript is disabled or hasn't loaded yet.
2.  **Type Safety:** The function is part of your server code, so you get full TypeScript support across the wire.
3.  **Automatic Revalidation:** You can call `revalidatePath('/')` inside the action to tell Next.js to refresh the UI immediately after the database update.

---

### 9. Next.js Middleware & The Edge Runtime

**Definition:** Middleware runs **before** a request is processed. It runs on the "Edge" (mini-servers distributed globally), making it extremely fast.

**Use Cases:**
-   **Auth Checks:** Redirecting unauthenticated users before they even hit your server.
-   **A/B Testing:** Serving different versions of a page based on a cookie.
-   **Geolocation:** Redirecting users based on their country.
-   **Bot Protection:** Blocking malicious IPs at the edge.

---

### 10. Memory Leaks in React

**Culprits:**
1.  **Uncleared Event Listeners:** Attaching `window.addEventListener` in `useEffect` and forgetting to return a cleanup function.
2.  **Uncleared Intervals/Timeouts:** `setInterval` that keeps running after the component is unmounted.
3.  **Subscription leaks:** Not unsubscribing from WebSockets or Firebase listeners.
4.  **Closure Leaks:** Large objects stuck in a closure that is still referenced by a stale timer.

---

### 11. State Management: When to use what?

-   **Context API:** Best for "Low Frequency" updates like Theme (Dark/Light) or User Auth. 
    -   *Problem:* Causes a re-render of **all** children when the value changes, making it slow for high-frequency data (like a real-time股票价格).
-   **Redux (Toolkit):** Best for massive logic-heavy enterprise apps with complex "Time Travel" debugging needs. (High boilerplate).
-   **Zustand (Recommended):** Extremely lightweight, zero boilerplate, and prevents unnecessary re-renders automatically. The modern mid-level standard.

---

### 12. React 18: `useSyncExternalStore`

**Problem:** If you have data living outside of React (e.g., in a vanilla JS store or a browser API like `window.devicePixelRatio`), how do you make React "react" to its changes safely in Concurrent Mode?
**Solution:** `useSyncExternalStore`. It ensures that your UI stays consistent with the external source of truth without "tearing" (UI glitches during concurrent rendering).

---

### 13. Error Boundaries: Why they fail for Async Errors

**Definition:** Error boundaries catch errors during **rendering** (in lifecycle methods or constructors).
**The Trap:** They do **NOT** catch errors in:
1.  Event handlers (clicks).
2.  Asynchronous code (`fetch`, `setTimeout`).
3.  Server-side rendering.
**Fix:** You must use `try/catch` inside asyc functions and either use a local error state or manually throw the error into the "render phase" to be caught.

---

### 14. Styling in Next.js: TailWind vs Styled Components

-   **Styled Components (Runtime CSS-in-JS):** Requires a JS library to compute styles on every render.
    -   *Big Cons:* Extremely difficult to use with **Server Components** because they rely on runtime context.
-   **Tailwind CSS (Zero Runtime):** Generates static CSS at build time.
    -   *Benefit:* 100% compatible with Server Components and highly optimized for performance.

---

### 15. Internationalization (i18n) in Next.js

Mid-level developers use **Middleware-based routing**.
1.  Detect user language from the `Accept-Language` header in Middleware.
2.  Redirect to `/[locale]/page`.
3.  Use "Server-side Dictionaries" (JSON files) that are loaded into Server Components. 
**Benefit:** Zero extra JS on the client for translation logic.

---

### 16. Testing Strategy: RTL vs Cypress

-   **React Testing Library (RTL):** Focuses on "Testing as a User". (e.g., "Click the button with text 'Login'"). It mocks the DOM but doesn't run a real browser.
-   **Cypress / Playwright:** Runs a **Real Browser**. Essential for testing Auth flows, File Uploads, and Cross-page navigation.
**Mid-Level Goal:** 70% RTL (Unit/Integration) and 20% Cypress (Critical E2E flows).

---

### 17. Security: `dangerouslySetInnerHTML`

**Problem:** How to render HTML strings without being hacked?
**The Rule:** Never trust user input. If you must use this prop, you **must** use a library like `DOMPurify` to "Sanitize" the HTML, stripping out `<script>` and `onclick` attributes.

---

### 18. `useLayoutEffect` vs `useEffect`

-   **`useEffect`**: Runs **after** the browser has painted the screen. Best for 99% of use cases (API calls, logging).
-   **`useLayoutEffect`**: Runs **synchronously** after DOM changes but **before** the browser paints the screen.
-   **Use Case:** Measuring DOM elements (e.g., finding the width of a tooltip) to prevent "flickering" before the final UI is shown.

---

### 19. Web Workers in React

**Scenario:** You have a massive JS calculation (e.g., processing a 10MB CSV in the browser).
-   If you do it on the main thread, the UI freezes.
-   **Solution:** Move the code to a **Web Worker**. The Worker runs in a separate background thread and sends the results back via `postMessage`.

---

### 20. App Router: Parallel & Intercepting Routes

-   **Parallel Routes:** Rendering two different pages at the exact same location (e.g., a `@dashboard` and `@feed` side-by-side).
-   **Intercepting Routes:** Showing a "Modal" version of a page when clicking a link, but showing the "Full Page" version on a direct URL refresh. (The "Soft Navigation" vs "Hard Navigation" pattern).

---

*Expansion Complete*
