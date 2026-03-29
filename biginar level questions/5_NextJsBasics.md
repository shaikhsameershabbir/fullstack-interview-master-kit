# Next.js Basics – Complete Beginner Interview Reference

---

## ⚡ Section 1: Rendering & Data Fetching

---

### 1. What is Next.js?

**Definition:** **Next.js** is a powerful **open-source React framework** created by Vercel that allows developers to build high-performance, SEO-friendly web applications by providing features like **Server-Side Rendering (SSR)** and **Static Site Generation (SSG)** out of the box.

**Key Features:**
-   **Hybrid Rendering:** Choose between SSR, SSG, CSR, and ISR on a per-page basis.
-   **Zero Config:** Automatic compilation and bundling (uses Webpack and Babel/SWC).
-   **File-based Routing:** Folders and files in the `pages` (or `app`) directory automatically become routes.
-   **API Routes:** Build backend API endpoints within the same project.
-   **Image Optimization:** Built-in `next/image` component for faster loading.
-   **Middleware:** Run code before a request is completed.

**Why use it over plain React?**
-   **SEO:** Search engines can easily crawl content since it's rendered on the server.
-   **Performance:** Faster First Contentful Paint (FCP).
-   **Developer Experience:** Built-in routing, image handling, and deployment optimizations.

---

### 2. What is SSR (Server-Side Rendering)?

**Definition:** In **SSR**, the HTML of a page is generated on the **server for every single request**. When a user visits a URL, the server fetches the data, renders the React components into HTML, and sends the full page back to the browser.

**How it works:**
1.  User requests a page.
2.  Server fetches data and generates HTML.
3.  Browser receives the full HTML (visible content).
4.  React "hydrates" the page (makes it interactive).

**Key Function:** `getServerSideProps`

**Best for:** Pages with dynamic data that changes frequently (e.g., a user profile, personalized dashboard, or live feed) where SEO is important.

---

### 3. What is CSR (Client-Side Rendering)?

**Definition:** **CSR** is the standard React approach. The server sends a nearly empty HTML file and a large JavaScript bundle to the browser. The browser then executes the JS, fetches data from an API, and renders the UI.

**Pros:**
-   Fast transitions after the initial load.
-   Reduced server load (rendering happens on the client).

**Cons:**
-   **Bad SEO:** Search engines might see an empty page before JS loads.
-   **Initial Delay:** User sees a blank screen or loader while JS/Data is being fetched.

**How to use in Next.js:** Simply use standard React `useEffect` and `fetch` within a component, or use the `'use client'` directive in the App Router.

---

### 4. What is SSG (Static Site Generation)?

**Definition:** In **SSG**, the HTML is generated **at build time** (when you run `npm run build`). The same pre-rendered HTML is then reused for every request and can be cached by a CDN.

**Pros:**
-   **Extremely Fast:** Pages are served as static files.
-   **Great SEO:** Full content is available immediately.

**Cons:**
-   Data can become stale (requires a rebuild to update).

**Key Function:** `getStaticProps` (and `getStaticPaths` for dynamic routes).

**Best for:** Pages that don't change often, like marketing pages, blog posts, documentation, or help centers.

---

### 5. What is ISR (Incremental Static Regeneration)?

**Definition:** **ISR** allows you to update static pages **after** you’ve built your site, without needing to rebuild the entire project. It combines the speed of SSG with the freshness of SSR.

**How it works:**
You specify a `revalidate` time (in seconds). After that time, if a request comes in, Next.js will trigger a background regeneration of the page.

**Example:**
```js
export async function getStaticProps() {
  const data = await fetchData();
  return {
    props: { data },
    revalidate: 60, // Regenerate page at most once every 60 seconds
  };
}
```

---

### 6. What is `getServerSideProps`?

**Definition:** A special function you export from a page to fetch data on **every request** (SSR).

**Code Example:**
```jsx
export async function getServerSideProps(context) {
  // 'context' contains params, req, res, query, etc.
  const res = await fetch(`https://api.example.com/data`);
  const data = await res.json();

  return {
    props: { data }, // passed to the page component as props
  };
}

export default function Page({ data }) {
  return <div>{data.title}</div>;
}
```

**Note:** This function only runs on the server.

---

### 7. What is `getStaticProps`?

**Definition:** A function you export to fetch data **at build time** (SSG).

**Code Example:**
```jsx
export async function getStaticProps() {
  const res = await fetch('https://api.example.com/posts');
  const posts = await res.json();

  return {
    props: { posts },
  };
}
```

**`getStaticPaths`:** Used alongside `getStaticProps` for dynamic routes (e.g., `/posts/[id]`) to tell Next.js which paths should be pre-rendered at build time.

---

### 8. What is Dynamic Routing?

**Definition:** Dynamic routing allows you to create pages where the URL contains variable parts (e.g., `/user/[id]`).

**Implementation:**
-   **Pages Router:** Create a file named `pages/posts/[id].js`.
-   **App Router:** Create a folder named `app/posts/[id]/page.js`.

The `id` can be accessed via `useRouter` (client) or `params` prop (server).

---

### 9. What are API Routes in Next.js?

**Definition:** API routes allow you to build a **Node.js backend** directly inside your Next.js app. Any file inside `pages/api` (or `app/api`) is mapped to `/api/*` and treated as an API endpoint.

**Example (`pages/api/hello.js`):**
```js
export default function handler(req, res) {
  if (req.method === 'POST') {
    // Handle POST request
  }
  res.status(200).json({ name: 'John Doe' });
}
```

---

### 10. What is Middleware in Next.js?

**Definition:** **Middleware** allows you to run code **before** a request is completed. Based on the incoming request, you can rewrite, redirect, modify the request/response headers, or respond directly.

**Common Use Cases:**
-   Authentication checks.
-   Redirecting based on geo-location or language.
-   A/B Testing.
-   Bot protection.

**File:** Located at `middleware.js` in the root (or `src`) directory.

---
## 🛤️ Section 2: Advanced Routing & Components

---

### 11. What is the App Router vs Pages Router?

**Pages Router (Legacy/Classic):**
-   Uses the `pages/` directory.
-   Routes are defined by file names.
-   Data fetching: `getStaticProps`, `getServerSideProps`.

**App Router (Modern/Recommended):**
-   Uses the `app/` directory (introduced in Next 13.4).
-   Supports **React Server Components (RSC)**.
-   Uses **Nested Layouts**, Loading states, and Error boundaries (`layout.js`, `loading.js`, `error.js`).
-   Data fetching: Standard `async/await` in components (Server Components).

---

### 12. What are React Server Components (RSC)?

**Definition:** **Server Components** are a new way of building components that **only render on the server**, and their code is never sent to the client.

**Key Difference:**
-   **Server Components:** Can access data directly (DB, file system) without an API. They have **zero bundle size** on the client.
-   **Client Components:** (Marked with `'use client'`). Regular React components that can use state (`useState`), effects (`useEffect`), and browser APIs.

---

### 13. What is the `Link` component in Next.js?

**Definition:** The **`<Link>`** component is used for **client-side navigation** between pages. It is more performant than a standard anchor tag (`<a>`).

**Key Advantages:**
-   **No Full Page Reload:** It only updates the content that changes.
-   **Prefetching:** Next.js automatically starts downloading the destination page (only the static parts) when the link appears in the viewport, making transitions feel instant.

---

### 14. What is Image Optimization (`next/image`)?

**Definition:** The **`<Image>`** component is a built-in feature that automatically optimizes images for performance.

**Features:**
-   **Lazy Loading:** Images only load when they enter the viewport.
-   **Responsive:** Automatically serves the correct image size for the user's screen (mobile vs desktop).
-   **Format Conversion:** Serves modern formats like **WebP** if the browser supports it.
-   **Resizing:** Prevents **Layout Shift** (CLS) by requiring dimensions (`width` and `height`).

---

### 15. What is the difference between `_app.js` and `_document.js`?

*(Note: These apply to the **Pages Router**)*

| File | Purpose | Key Use Cases |
|---|---|---|
| **`_app.js`** | Wraps every page. Used to initialize pages. | Global CSS, keeping state during navigation, Layouts. |
| **`_document.js`** | Used to augment the application's `<html>` and `<body>` tags. | Custom fonts (Google Fonts), meta tags that apply globally, third-party script tags. |

---

### 16. How to handle SEO in Next.js?

**Pages Router:**
Use the **`<Head>`** component provided by `next/head`.
```jsx
import Head from 'next/head';

function MyPage() {
  return (
    <div>
      <Head>
        <title>My Page Title</title>
        <meta name="description" content="A great description" />
      </Head>
      <h1>Content</h1>
    </div>
  );
}
```

**App Router:**
Use the **Metadata API** by exporting a `metadata` object or `generateMetadata` function from a `layout.js` or `page.js`.

---

### 17. What is the `next/font` component?

**Definition:** A built-in feature that automatically **optimizes and self-hosts fonts**. It works with Google Fonts and local fonts.

**Benefits:**
-   **No Layout Shift:** Zero "flash of unstyled text" (FOUT).
-   **Performance:** Fonts are downloaded at build time; no external requests to Google are made at runtime.
-   **Privacy:** Since fonts are self-hosted, user data isn't sent to font providers.

---

### 18. How do you handle Environment Variables in Next.js?

**Definition:** Store sensitive data (like API keys) in a `.env` file.
Next.js supports multiple environments (`.env.local`, `.env.development`, `.env.production`).

**Rules:**
-   **Server-only:** By default, variables are only available on the server (node.js).
-   **Client-side access:** Prefix the variable with **`NEXT_PUBLIC_`** to make it accessible to the browser.
    -   Example: `NEXT_PUBLIC_API_URL`, `STRIPE_SECRET_KEY`.

---

### 19. What is Prefetching in Next.js?

**Definition:** **Prefetching** is the process where Next.js automatically starts downloading pages in the background as soon as a `<Link>` component enters the user's viewport.

**How it works:**
-   It only prefetches the **code bundle** (JS) and the results of `getStaticProps`.
-   This makes clicking the link feel like an **instant** transition.
-   It only happens on **production** builds.

---

### 20. How do you implement Dynamic Metadata in App Router?

**Implementation:**
Export a **`generateMetadata`** function that can fetch data (e.g., product details) and return dynamic SEO tags.

```js
export async function generateMetadata({ params }) {
  const product = await fetchProduct(params.id);
  return {
    title: product.name,
    description: product.summary,
  };
}
```

---

## 🚀 Section 3: Optimization & Best Practices

---

### 21. What is "Layouts" in Next.js?

**Definition:** **Layouts** are UI components shared across multiple pages. They allow you to maintain state across navigation (e.g., a navbar that stays persistent).

**App Router (`layout.js`):**
-   Layouts are nested. A layout for a folder applies to all segments in that folder.
-   Unlike pages, layouts **do not re-render** when navigating.

---

### 22. What is "Streaming" in Next.js?

**Definition:** **Streaming** allows you to break down the HTML of a page into smaller chunks and progressively send them from the server to the client. This means parts of the page (like a navbar) can be displayed immediately while slower, data-heavy parts (like a product list) are still loading.

**How to implement:**
-   Use **`loading.js`** file in the App Router.
-   Use **React `<Suspense>`** boundaries.

---

### 23. What is the difference between `next build` and `next dev`?

-   **`next dev`:** Starts a development server with **Hot Module Replacement (HMR)** and error reporting. It doesn't optimize code for performance.
-   **`next build`:** Compiles the application for **production**. It generates the specialized HTML files (SSG), optimizes JS bundles, and minifies CSS.
-   **`next start`:** Runs the production-ready server (after `next build`).

---

### 24. What is "Vercel"?

**Definition:** **Vercel** is the platform that created Next.js. It provides a cloud infrastructure designed to host frontend frameworks and static sites with **zero configuration**.

**Features:**
-   **Automatic Deployments:** Every git push triggers a new deployment.
-   **Global CDN:** Automatically serves your static files and images from the edge.
-   **Serverless Functions:** Handles API routes automatically as serverless functions.
-   **Preview Deployments:** Each pull request gets its own unique URL for testing.

---

### 25. What is the purpose of `next.config.js`?

**Definition:** This file is used to provide **custom configuration** for your Next.js project.

**Common uses:**
-   Adding environment variables.
-   Configuring allowed image domains (for `next/image`).
-   Adding webpack custom configurations.
-   Setting up redirects or rewrites.
-   Enabling experimental features.

---

### 26. What is the "Cache" in Next.js 13/14+?

**Definition:** The App Router introduced a complex caching system to minimize external requests.
1.  **Request Memoization:** Reuses data across a single request (e.g., calling `fetch` twice with same URL only calls it once).
2.  **Data Cache:** Persists data across multiple user requests.
3.  **Full Route Cache:** Stores the rendered HTML/RSC of a page.
4.  **Router Cache:** Client-side cache that stores visited segments.

---

### 27. How to handle "404 - Not Found" in Next.js?

**Pages Router:**
Create a file named `pages/404.js`. Next.js automatically shows this for any missing route.

**App Router:**
Create a file named `app/not-found.js`. You can also trigger it manually using the **`notFound()`** function.

---

### 28. What is "Static Export"?

**Definition:** Next.js can be configured to export every page as a literal **static HTML/CSS/JS file** (no Node.js server required).

**How to enable:**
Add `output: 'export'` in your `next.config.js`.

**Use case:** Hosting on services like GitHub Pages, S3, or any static host. (Note: API routes and SSR won't work in this mode).

---

### 29. What is "Shallow Routing"?

**Definition:** In the **Pages Router**, shallow routing allows you to change the URL without running data fetching methods like `getServerSideProps` or `getStaticProps` again.

**Usage:**
```js
router.push('/?counter=10', undefined, { shallow: true });
```
*(Note: Only works for URL changes on the same page).*

---

### 30. Common Next.js Best Practices Checklist:

```
✅ Use SSG (getStaticProps) by default for better performance.
✅ Use SSR (getServerSideProps) only when data is highly dynamic/personalized.
✅ Prefer components over pages for complex UI.
✅ Use next/image for ALL images.
✅ Use next/link for all internal navigation.
✅ Optimize SEO using Metadata API (App) or Head component (Pages).
✅ Keep your .env files secure and never commit them to git.
✅ Leverage ISR (Incremental Static Regeneration) for large content sites (e.g., Blogs/E-commerce).
✅ Use the App Router for new projects to take advantage of Server Components.
```

---

*End of Next.js Basics Reference*

---

### 31. What is the difference between `<Link>` and `useRouter()`?

-   **`<Link>` component:** The recommended way to navigate. It handles prefetching automatically and is more SEO-friendly because it renders as an `<a>` tag.
-   **`useRouter()` hook:** A programmatic way to navigate (e.g., `router.push('/dashboard')`). It's used inside event handlers or `useEffect` when you need to navigate based on logic (like after a form submission).

---

### 32. How to handle "Not Found" (404) in App Router?

In the App Router, you can create a special file named **`not-found.js`** inside any segment of your application. Next.js will automatically show this UI when the **`notFound()`** function is triggered or when a route doesn't exist.

---

### 33. What is the `usePathname` Hook?

**Definition:** A Client Component hook that lets you read the current URL's **pathname**. It's useful for highlighting the "active" link in a navigation bar.

```js
'use client'
import { usePathname } from 'next/navigation'

const pathname = usePathname() // returns e.g., "/about"
```

---

### 34. What is the `useSearchParams` Hook?

**Definition:** A Client Component hook that lets you read the **query parameters** from the current URL.

```js
'use client'
import { useSearchParams } from 'next/navigation'

const searchParams = useSearchParams()
const search = searchParams.get('search') // gets ?search=abc
```

---

### 35. What is the Metadata API in Next.js?

**Definition:** Next.js has a Metadata API that can be used to define your application metadata (e.g. `title`, `description`, `openGraph` tags) for improved SEO.

**Static Metadata:**
```js
export const metadata = {
  title: 'My Page',
};
```

**Dynamic Metadata:**
```js
export async function generateMetadata({ params }) {
  return { title: `Product ${params.id}` };
}
```

---

### 36. What are Parallel Routes?

**Definition:** Parallel Routes allow you to simultaneously or conditionally render one or more pages within the same layout. They are defined using "slots" (folders starting with `@`).

---

### 37. What are Intercepting Routes?

**Definition:** Intercepting routes allow you to load a route from another part of your application within the current layout. This is commonly used for showing a **modal** when clicking an image, but having a full page when the URL is shared or refreshed.

---

### 38. What is `generateStaticParams`?

**Definition:** In the App Router, `generateStaticParams` is used to **statically generate routes** at build time for dynamic segments (like `/blog/[slug]`). It replaces `getStaticPaths` from the Pages Router.

---

### 39. What are Server Actions?

**Definition:** **Server Actions** are asynchronous functions that run on the server. They can be invoked directly from Client or Server Components to handle form submissions and data mutations.

```js
async function createInvoice(formData) {
  'use server'
  // Logic to save to DB
}
```

---

### 40. How to deploy a Next.js app on Vercel?

1.  Connect your GitHub/GitLab repository to Vercel.
2.  Vercel automatically detects Next.js and configures the build settings.
3.  Each `git push` triggers a new deployment.
4.  Vercel handles SSL, Global CDN, and Serverless Functions automatically.

---

*End of Next.js Basics Reference*
