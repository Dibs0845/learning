# React & Next.js — Conceptual Questions & Answers

A guide to conceptual interview and learning questions, with simple explanations and small examples.

---

## Table of Contents

**React**
1. [React Basics](#1-react-basics)
2. [JSX](#2-jsx)
3. [Components & Props](#3-components--props)
4. [State & useState](#4-state--usestate)
5. [Event Handling](#5-event-handling)
6. [Conditional & List Rendering](#6-conditional--list-rendering)
7. [Hooks](#7-hooks)
8. [useEffect & Lifecycle](#8-useeffect--lifecycle)
9. [Refs](#9-refs)
10. [Context API](#10-context-api)
11. [Performance Optimization](#11-performance-optimization)
12. [Forms](#12-forms)
13. [Advanced Concepts](#13-advanced-concepts)

**Next.js**
14. [Next.js Basics](#14-nextjs-basics)
15. [Routing](#15-routing)
16. [Rendering Strategies (SSR, SSG, ISR, CSR)](#16-rendering-strategies)
17. [App Router & Server Components](#17-app-router--server-components)
18. [Data Fetching](#18-data-fetching)
19. [API Routes / Route Handlers](#19-api-routes--route-handlers)
20. [Optimization Features](#20-optimization-features)
21. [Deployment & Config](#21-deployment--config)

---

# REACT

## 1. React Basics

### Q1. What is React?
React is a **JavaScript library** (not a full framework) for building user interfaces, created by Facebook/Meta. It lets you build UIs out of small, reusable pieces called **components**. Its core idea: describe *what* the UI should look like for a given state, and React figures out *how* to update the DOM.

### Q2. Why use React? What problems does it solve?
- **Component-based**: Break complex UIs into small, reusable, testable pieces.
- **Declarative**: You describe the end result; React handles DOM manipulation.
- **Efficient updates**: The Virtual DOM minimizes expensive real-DOM operations.
- **Large ecosystem**: Huge community, tools, and libraries.

### Q3. What is the Virtual DOM?
The Virtual DOM is a **lightweight JavaScript copy of the real DOM** kept in memory. When state changes:
1. React builds a new Virtual DOM tree.
2. It **diffs** the new tree against the previous one (a process called **reconciliation**).
3. It applies only the **minimal set of changes** to the real DOM.

The real DOM is slow to update; the Virtual DOM lets React batch and minimize those updates.

### Q4. What is reconciliation?
Reconciliation is the algorithm React uses to compare the previous and current Virtual DOM trees and decide what changed. Keys and element types help React decide whether to reuse or recreate DOM nodes.

### Q5. What is the difference between a library and a framework?
A **library** (React) gives you tools to do one thing (build UI); *you* control the flow. A **framework** (Angular, Next.js) provides structure and calls *your* code (inversion of control). React is often paired with routing/state libraries to form a full stack.

### Q6. What is a Single Page Application (SPA)?
An SPA loads a single HTML page and dynamically updates content via JavaScript as the user navigates — no full page reloads. React is commonly used to build SPAs.

---

## 2. JSX

### Q7. What is JSX?
JSX is a **syntax extension** for JavaScript that lets you write HTML-like markup inside JS. It is not understood by browsers directly — tools like Babel **transpile** it into `React.createElement()` calls.

```jsx
const element = <h1>Hello</h1>;
// becomes:
const element = React.createElement('h1', null, 'Hello');
```

### Q8. What are the rules of JSX?
- Return a **single root element** (or use a Fragment `<>...</>`).
- Use `className` instead of `class`, `htmlFor` instead of `for`.
- Close all tags, including self-closing ones (`<img />`).
- Use camelCase for most attributes (`onClick`, `tabIndex`).
- Embed JavaScript expressions with curly braces: `{expression}`.

### Q9. What is a React Fragment and why use it?
A Fragment lets you group multiple children without adding an extra DOM node.

```jsx
<>
  <td>Hello</td>
  <td>World</td>
</>
```
Useful when extra `<div>` wrappers would break layout (e.g., table rows, flex/grid).

### Q10. Can you use if/else directly inside JSX?
No — JSX accepts **expressions**, not statements. Use ternaries (`cond ? a : b`), logical AND (`cond && a`), or compute the value before the `return`.

---

## 3. Components & Props

### Q11. What are the types of components?
- **Function components** (modern standard): plain JS functions returning JSX.
- **Class components** (legacy): ES6 classes extending `React.Component`.

```jsx
function Welcome({ name }) {
  return <h1>Hello, {name}</h1>;
}
```

### Q12. What are props?
Props (properties) are **read-only inputs** passed from a parent to a child component, like function arguments. They make components reusable and configurable.

```jsx
<Welcome name="Alice" />   // name is a prop
```

### Q13. Can a child modify its props?
No. Props are **immutable** from the child's perspective. Data flows one-way (parent → child). To change data, the parent must pass new props, or the child calls a callback function passed via props.

### Q14. What is "one-way data flow"?
Data flows **downward** from parent to child through props. This makes apps predictable — you always know where data comes from. To send data upward, children call functions (callbacks) that the parent provided.

### Q15. What are `children` props?
`children` is a special prop containing whatever you nest between a component's tags.

```jsx
function Card({ children }) {
  return <div className="card">{children}</div>;
}
<Card><p>Inside</p></Card>   // <p> is children
```

### Q16. What is prop drilling?
Passing props through many intermediate components that don't use them, just to reach a deeply nested child. It's tedious and fragile. Solved with **Context API** or state libraries (Redux, Zustand).

### Q17. What are default props?
Default values used when a prop isn't passed:
```jsx
function Button({ type = "button" }) { ... }
```

---

## 4. State & useState

### Q18. What is state?
State is **data managed inside a component** that can change over time. When state changes, React **re-renders** the component to reflect the new data. Unlike props, state is local and mutable (via setter functions).

### Q19. What is the difference between state and props?
| Props | State |
|-------|-------|
| Passed from parent | Managed within component |
| Read-only (immutable) | Changeable (via setter) |
| Configure a component | Track changing data |

### Q20. How does useState work?
`useState` returns an array: the current value and a setter function.

```jsx
const [count, setCount] = useState(0);
setCount(count + 1);   // triggers a re-render
```

### Q21. Why is state updated asynchronously / in batches?
React **batches** multiple state updates in an event handler for performance, applying them together and re-rendering once. That's why reading state right after setting it shows the old value.

### Q22. What is the functional update form and when do you need it?
When the new state depends on the previous state, pass a function:
```jsx
setCount(prev => prev + 1);
```
This guarantees you use the latest value, avoiding bugs from stale/batched state.

### Q23. Why must you not mutate state directly?
React detects changes by **comparing references** (shallow comparison). Mutating an object/array in place keeps the same reference, so React may not re-render. Always create a new object/array:
```jsx
setItems([...items, newItem]);           // ✅
setUser({ ...user, name: "New" });       // ✅
items.push(newItem); setItems(items);    // ❌ same reference
```

### Q24. What is lifting state up?
When two sibling components need to share state, you move ("lift") the state to their nearest **common parent**, then pass it down via props. This keeps a single source of truth.

### Q25. What is a controlled vs uncontrolled component?
- **Controlled**: form input value is driven by React state (`value` + `onChange`). React is the single source of truth.
- **Uncontrolled**: the DOM holds the value; you read it via a ref. Less code, less control.

---

## 5. Event Handling

### Q26. How does React handle events?
React uses **SyntheticEvents** — a cross-browser wrapper around native events for consistent behavior. You attach handlers with camelCase props: `onClick`, `onChange`, `onSubmit`.

```jsx
<button onClick={handleClick}>Click</button>
```

### Q27. Why pass a function reference, not call it?
```jsx
<button onClick={handleClick}>   // ✅ reference
<button onClick={handleClick()}> // ❌ calls immediately on render
```
Use an arrow function to pass arguments: `onClick={() => handleClick(id)}`.

### Q28. How do you prevent default browser behavior?
Call `e.preventDefault()` inside the handler (e.g., stopping a form from reloading the page). `e.stopPropagation()` stops the event from bubbling to parents.

---

## 6. Conditional & List Rendering

### Q28b. How do you render lists?
Use `.map()` to turn an array into JSX elements:
```jsx
{users.map(user => <li key={user.id}>{user.name}</li>)}
```

### Q29. Why are keys important in lists?
Keys give each list item a **stable identity** so React can efficiently track which items were added, removed, or reordered during reconciliation. Without correct keys, React may re-render incorrectly or lose component state.

### Q30. Why shouldn't you use array index as a key?
If the list can reorder, insert, or delete items, index-based keys cause React to associate the wrong state/DOM with items, leading to bugs. Use a **stable unique ID** instead. (Index is acceptable only for static, never-changing lists.)

---

## 7. Hooks

### Q31. What are Hooks?
Hooks are functions (introduced in React 16.8) that let function components "hook into" React features like state and lifecycle — without writing classes. Examples: `useState`, `useEffect`, `useContext`, `useRef`, `useMemo`, `useCallback`, `useReducer`.

### Q32. What are the Rules of Hooks?
1. **Only call Hooks at the top level** — not inside loops, conditions, or nested functions. (React relies on call order.)
2. **Only call Hooks from React function components or custom Hooks** — not regular JS functions.

### Q33. What is a custom Hook?
A reusable function starting with `use` that combines built-in hooks to share **stateful logic** across components.
```jsx
function useToggle(initial = false) {
  const [on, setOn] = useState(initial);
  const toggle = () => setOn(o => !o);
  return [on, toggle];
}
```

### Q34. What is useReducer and when to use it?
`useReducer` manages complex state logic via a reducer function `(state, action) => newState`. Prefer it over `useState` when state has multiple sub-values or the next state depends on complex transitions.
```jsx
const [state, dispatch] = useReducer(reducer, initialState);
dispatch({ type: 'increment' });
```

### Q35. What is the difference between useMemo and useCallback?
- **useMemo** memoizes a **computed value** — recomputes only when dependencies change.
- **useCallback** memoizes a **function reference** — returns the same function unless dependencies change.
```jsx
const total = useMemo(() => expensiveCalc(items), [items]);
const handler = useCallback(() => doThing(id), [id]);
```
`useCallback(fn, deps)` is essentially `useMemo(() => fn, deps)`.

### Q36. Why would you memoize a function with useCallback?
To keep a **stable reference** so that child components wrapped in `React.memo` don't re-render unnecessarily, or to keep `useEffect` dependencies stable.

---

## 8. useEffect & Lifecycle

### Q37. What is useEffect?
`useEffect` runs **side effects** — code that interacts with the outside world (data fetching, subscriptions, timers, manual DOM changes) — after render.

```jsx
useEffect(() => {
  // effect
  return () => { /* cleanup */ };
}, [dependencies]);
```

### Q38. How does the dependency array work?
- **No array**: runs after **every** render.
- **Empty `[]`**: runs **once** after the first render (mount).
- **`[a, b]`**: runs after mount and whenever `a` or `b` changes.

### Q39. What is the cleanup function?
The function you **return** from an effect. React runs it before the next effect run and on unmount. Use it to unsubscribe, clear timers, cancel requests — preventing memory leaks.
```jsx
useEffect(() => {
  const id = setInterval(tick, 1000);
  return () => clearInterval(id);
}, []);
```

### Q40. How do class lifecycle methods map to useEffect?
- `componentDidMount` → `useEffect(() => {}, [])`
- `componentDidUpdate` → `useEffect(() => {}, [deps])`
- `componentWillUnmount` → the cleanup `return () => {}`

### Q41. What is a common useEffect infinite loop cause?
Updating state inside an effect that also lists that state (or an object recreated each render) as a dependency. Each update re-runs the effect, which updates state again, forever. Fix dependencies or use functional updates.

### Q42. What is useLayoutEffect and how does it differ?
`useLayoutEffect` runs **synchronously after DOM mutations but before the browser paints**. Use it for DOM measurements to avoid visual flicker. `useEffect` runs after paint (asynchronously) and is preferred for most cases.

---

## 9. Refs

### Q43. What is useRef used for?
`useRef` returns a mutable object `{ current: value }` that **persists across renders without causing re-renders**. Two main uses:
1. **Access DOM elements** directly (focus, scroll, measure).
2. **Store mutable values** (like a previous value or a timer ID) that shouldn't trigger renders.

```jsx
const inputRef = useRef(null);
<input ref={inputRef} />
inputRef.current.focus();
```

### Q44. Difference between state and ref?
Changing state **re-renders**; changing a ref **does not**. Use state for data that affects the UI, refs for values that don't.

### Q45. What is forwardRef?
`forwardRef` lets a parent pass a ref down to a DOM node inside a child component, since refs aren't normal props. (In React 19, `ref` can be passed as a regular prop, reducing the need for `forwardRef`.)

---

## 10. Context API

### Q46. What is the Context API?
Context provides a way to share data (theme, auth user, language) across the component tree **without prop drilling**. Steps: create context, wrap tree with a `Provider`, consume with `useContext`.

```jsx
const ThemeContext = createContext('light');

<ThemeContext.Provider value="dark">
  <App />
</ThemeContext.Provider>

const theme = useContext(ThemeContext);
```

### Q47. When should you NOT use Context?
Context isn't a full state manager. Overusing it (especially for frequently-changing values) causes **all consumers to re-render**. For large/complex global state, use Redux, Zustand, or Jotai. Context is best for low-frequency, truly global data.

### Q48. Context vs Redux?
Context is a **transport mechanism** for passing data; Redux is a **state management library** with a store, actions, reducers, middleware, and devtools. Use Context for simple sharing, Redux for complex, large-scale state with predictable updates.

---

## 11. Performance Optimization

### Q49. What is React.memo?
A higher-order component that **memoizes a component**, skipping re-render if its props haven't changed (shallow comparison). Useful for expensive components that receive the same props often.
```jsx
const MyComp = React.memo(function MyComp(props) { ... });
```

### Q50. What causes unnecessary re-renders?
- Parent re-renders (children re-render by default).
- New object/array/function references passed as props each render.
- Context value changes.
Fix with `React.memo`, `useMemo`, `useCallback`, and stable references.

### Q51. What is code splitting / lazy loading?
Splitting your bundle so code loads **on demand** rather than all upfront, improving initial load time. Use `React.lazy` + `Suspense`:
```jsx
const Chart = React.lazy(() => import('./Chart'));
<Suspense fallback={<Spinner />}><Chart /></Suspense>
```

### Q52. What is Suspense?
`Suspense` lets you show a **fallback UI** (like a loader) while a lazy component or async data is loading.

### Q53. What is windowing / virtualization?
Rendering only the **visible portion** of a long list (e.g., with react-window/react-virtualized) instead of thousands of DOM nodes, dramatically improving performance.

### Q54. What is the key mistake of premature optimization here?
Wrapping everything in `memo`/`useMemo`/`useCallback` adds overhead and complexity. Measure first (React DevTools Profiler), then optimize real bottlenecks.

---

## 12. Forms

### Q55. How do you handle a controlled form?
Bind each input to state:
```jsx
const [email, setEmail] = useState('');
<input value={email} onChange={e => setEmail(e.target.value)} />
```

### Q56. How do you handle multiple inputs efficiently?
Use one state object and the input's `name`:
```jsx
const [form, setForm] = useState({ name: '', email: '' });
const handle = e =>
  setForm({ ...form, [e.target.name]: e.target.value });
```

### Q57. Popular form libraries?
**React Hook Form** (performant, uncontrolled-based) and **Formik** — they handle validation, errors, and submission with less boilerplate.

---

## 13. Advanced Concepts

### Q58. What is a Higher-Order Component (HOC)?
A function that takes a component and returns a new enhanced component. Pattern for reusing logic (largely replaced by hooks).
```jsx
const withAuth = Component => props => <Component {...props} user={user} />;
```

### Q59. What is the render props pattern?
Sharing logic by passing a function as a prop (often `children`) that returns JSX.
```jsx
<DataProvider>{data => <List items={data} />}</DataProvider>
```

### Q60. What are Error Boundaries?
Components that **catch JavaScript errors** in their child tree, log them, and show a fallback UI instead of crashing the whole app. Currently only class components (with `componentDidCatch` / `getDerivedStateFromError`) can be error boundaries.

### Q61. What is StrictMode?
`<React.StrictMode>` is a dev-only tool that highlights potential problems — it intentionally **double-invokes** certain functions (renders, effects) to surface impure logic and side-effect bugs. No effect in production.

### Q62. What is the difference between React 18's concurrent features?
React 18 introduced **concurrent rendering**: React can pause, resume, and prioritize renders. Features: `useTransition` (mark updates as non-urgent), `useDeferredValue` (defer expensive updates), and automatic batching everywhere.

### Q63. What are keys' role in component identity/state?
Changing a component's `key` forces React to **unmount and remount** it, resetting its state. This is a handy trick to reset a component (e.g., reset a form when a key/id changes).

---

# NEXT.JS

## 14. Next.js Basics

### Q64. What is Next.js?
Next.js is a **React framework** built on top of React that adds production features out of the box: file-based routing, server-side rendering, static generation, API routes, image optimization, and bundling. Maintained by Vercel.

### Q65. Why use Next.js over plain React (CRA/Vite)?
- **Built-in routing** (no react-router setup).
- **SSR/SSG/ISR** for better SEO and performance.
- **API routes** — build backend endpoints in the same project.
- **Automatic code splitting, image/font optimization**.
- **Full-stack** capability (Server Components, Server Actions).

### Q66. What problems does SSR solve that a plain SPA has?
Plain SPAs ship a near-empty HTML and render on the client, which hurts **SEO** (crawlers may see empty pages) and **initial load / time-to-content**. SSR sends fully-rendered HTML, improving both.

---

## 15. Routing

### Q67. What is file-based routing?
In Next.js, the **file/folder structure defines routes** — no manual route config. A file at `pages/about.js` (Pages Router) or `app/about/page.js` (App Router) automatically becomes `/about`.

### Q68. What are dynamic routes?
Routes with parameters, defined with brackets:
- `app/blog/[slug]/page.js` → `/blog/hello`
- `[...slug]` → catch-all (`/a/b/c`)
- `[[...slug]]` → optional catch-all (matches `/` too)

### Q69. How do you navigate between pages?
Use the `<Link>` component for client-side navigation (no full reload):
```jsx
import Link from 'next/link';
<Link href="/about">About</Link>
```
Programmatically, use the `useRouter` hook: `router.push('/about')`.

### Q70. What is the difference between the Pages Router and the App Router?
- **Pages Router** (`pages/`): the original system; uses `getServerSideProps`, `getStaticProps`, `getInitialProps`.
- **App Router** (`app/`, Next.js 13+): newer; built on **React Server Components**, uses layouts, `loading.js`, `error.js`, Server Actions, and `fetch`-based data fetching. It's the recommended approach for new apps.

---

## 16. Rendering Strategies

### Q71. What are the main rendering strategies in Next.js?
1. **CSR (Client-Side Rendering)** — rendered in the browser (like a normal SPA).
2. **SSR (Server-Side Rendering)** — HTML generated on the server **per request**.
3. **SSG (Static Site Generation)** — HTML generated **at build time**.
4. **ISR (Incremental Static Regeneration)** — static pages that **regenerate** in the background after a set time.

### Q72. When to use SSG?
For content that doesn't change per request or often: blogs, docs, marketing pages. Fast (served as static files/CDN) and great for SEO.

### Q73. When to use SSR?
When the page needs **fresh, per-request data** or is personalized (e.g., dashboards, user-specific content, request headers/cookies).

### Q74. What is ISR and why is it powerful?
ISR gives you static speed **plus** freshness. Pages are built statically, but after a `revalidate` interval, Next.js regenerates them in the background on the next request. You get CDN performance without full rebuilds.
```js
// Pages Router
export async function getStaticProps() {
  return { props: {...}, revalidate: 60 }; // regenerate every 60s
}
```

### Q75. What is hydration?
After the server sends static HTML, React **attaches event handlers and makes it interactive** on the client — this is hydration. A mismatch between server and client output causes "hydration errors."

---

## 17. App Router & Server Components

### Q76. What are React Server Components (RSC)?
Components that **render on the server only**. They can fetch data directly, access backend resources, and **don't ship their JS to the client**, reducing bundle size. In the App Router, components are Server Components **by default**.

### Q77. What is a Client Component and how do you make one?
A component that runs in the browser (has interactivity, state, effects, browser APIs). Add the directive at the top of the file:
```jsx
'use client';
```
Use Client Components when you need `useState`, `useEffect`, event handlers, or browser-only APIs.

### Q78. Server vs Client Components — key differences?
| Server Component | Client Component |
|------------------|------------------|
| Runs on server | Runs in browser |
| No JS shipped to client | Ships JS |
| Can access DB/filesystem/secrets | Cannot |
| No state/effects/event handlers | Full hooks & interactivity |
| Default in App Router | Opt-in via `'use client'` |

### Q79. What are layouts in the App Router?
`layout.js` wraps pages and **persists across navigation** (doesn't re-render), ideal for shared UI like navbars/sidebars. Layouts nest — a root layout wraps everything, and nested layouts add per-section UI.

### Q80. What are the special files in the App Router?
- `page.js` — the route's UI.
- `layout.js` — shared wrapper.
- `loading.js` — automatic loading UI (uses Suspense).
- `error.js` — error boundary UI.
- `not-found.js` — 404 UI.
- `template.js` — like layout but re-renders on navigation.
- `route.js` — API endpoint (Route Handler).

### Q81. What are Server Actions?
Functions marked `'use server'` that run on the server and can be called from components (e.g., form submissions) **without manually creating an API route**. They enable mutations directly from the UI.
```jsx
async function createTodo(formData) {
  'use server';
  await db.insert(...);
}
```

---

## 18. Data Fetching

### Q82. How do you fetch data in the App Router?
Directly in **async Server Components** using `fetch` (or any async call):
```jsx
async function Page() {
  const res = await fetch('https://api.example.com/data');
  const data = await res.json();
  return <div>{data.title}</div>;
}
```
Next.js extends `fetch` with caching and revalidation controls.

### Q83. How does caching/revalidation work with fetch?
```js
fetch(url)                              // cached by default (like SSG)
fetch(url, { cache: 'no-store' })       // always fresh (like SSR)
fetch(url, { next: { revalidate: 60 }}) // ISR-like, revalidate every 60s
```

### Q84. What are the data-fetching functions in the Pages Router?
- **`getStaticProps`** — runs at build time (SSG).
- **`getServerSideProps`** — runs on every request (SSR).
- **`getStaticPaths`** — specifies which dynamic paths to pre-render for SSG.

### Q85. What is the difference between getStaticProps and getServerSideProps?
`getStaticProps` runs **once at build time** (fast, cached, static). `getServerSideProps` runs **on every request** (fresh data, slower, server work per request).

---

## 19. API Routes / Route Handlers

### Q86. What are API routes?
Backend endpoints inside your Next.js app. In the Pages Router, files in `pages/api/` export a handler `(req, res)`. In the App Router, `route.js` files export functions named after HTTP methods (`GET`, `POST`, etc.).

```js
// app/api/hello/route.js
export async function GET() {
  return Response.json({ message: 'Hello' });
}
```

### Q87. Why are API routes useful?
Keep secrets (API keys, DB credentials) on the server, build a backend without a separate server, proxy third-party APIs, handle webhooks and form submissions.

### Q88. What is middleware in Next.js?
`middleware.js` runs **before a request completes**, at the edge. Use it for auth checks, redirects, rewrites, geolocation, and setting headers/cookies — applied across matching routes.

---

## 20. Optimization Features

### Q89. What does next/image provide?
The `<Image>` component gives **automatic optimization**: resizing, modern formats (WebP/AVIF), lazy loading, and preventing layout shift (via width/height). Improves performance and Core Web Vitals.

### Q90. What does next/font do?
Self-hosts and optimizes fonts (Google or local) at build time, eliminating external requests and font layout shift, improving privacy and performance.

### Q91. What is automatic code splitting in Next.js?
Each page/route only loads the JS it needs. Next.js splits bundles per route automatically, so users don't download the whole app upfront.

### Q92. How do you handle environment variables?
Store in `.env.local`. Server-only vars are private by default. Variables prefixed with `NEXT_PUBLIC_` are **exposed to the browser** — never put secrets there.

### Q93. What is dynamic import in Next.js?
`next/dynamic` lets you lazy-load components (optionally client-only with `{ ssr: false }`):
```js
const Chart = dynamic(() => import('./Chart'), { ssr: false });
```

---

## 21. Deployment & Config

### Q94. How is a Next.js app typically deployed?
On **Vercel** (zero-config, made by the same team) or any Node host, or exported as a static site (`output: 'export'`) for static hosting when no server features are needed.

### Q95. What is the difference between `next build`, `next start`, and `next dev`?
- `next dev` — development server with hot reload.
- `next build` — production build (optimizes, pre-renders static pages).
- `next start` — runs the built production server.

### Q96. What are metadata and SEO features?
The App Router provides a `metadata` export (or `generateMetadata`) to set `<title>`, description, Open Graph tags, etc., per route — improving SEO without manual `<head>` management.
```js
export const metadata = { title: 'Home', description: 'Welcome' };
```

### Q97. What is the `public` folder?
Static assets (images, robots.txt, favicons) served from the root URL. A file at `public/logo.png` is available at `/logo.png`.

---

## Quick Comparison Cheat Sheet

| Concept | React | Next.js |
|---------|-------|---------|
| Type | UI library | React framework |
| Routing | Needs react-router | Built-in file-based |
| Rendering | CSR by default | CSR, SSR, SSG, ISR |
| Backend | None | API routes / Server Actions |
| SEO | Weak (SPA) | Strong (SSR/SSG) |
| Setup | Vite/CRA | `create-next-app` |

---

## Common "Explain the Difference" Questions

- **State vs Props** → internal & mutable vs external & read-only.
- **useEffect vs useLayoutEffect** → after paint vs before paint.
- **useMemo vs useCallback** → memoize value vs memoize function.
- **Server vs Client Components** → server-only/no-JS vs interactive/ships-JS.
- **SSG vs SSR vs ISR** → build time vs per-request vs periodic regeneration.
- **Controlled vs Uncontrolled** → React state vs DOM/ref.
- **Context vs Redux** → simple sharing vs full state management.
- **Pages Router vs App Router** → classic getX props vs Server Components + layouts.

---

# SCENARIO-BASED QUESTIONS

These are practical "what would you do" and "why is this happening" questions — the kind asked to test real understanding, not just definitions.

## React Scenarios

### S1. "My component isn't re-rendering when I update an array in state. Why?"
You're most likely **mutating** the array instead of creating a new one. React compares by reference, so pushing into the same array keeps the same reference and skips the re-render.
```jsx
// ❌ Bug — same reference
todos.push(newTodo);
setTodos(todos);

// ✅ Fix — new reference
setTodos([...todos, newTodo]);
```

### S2. "I call setCount twice in one handler but it only increments by 1. Why?"
State updates are **batched**, and both calls read the same stale `count`. Use the functional updater so each call sees the latest value.
```jsx
// ❌ Both read count = 0 → ends at 1
setCount(count + 1);
setCount(count + 1);

// ✅ Ends at 2
setCount(c => c + 1);
setCount(c => c + 1);
```

### S3. "My useEffect runs in an infinite loop. What's wrong?"
An effect updates state that (directly or indirectly) is in its own dependency array. Common culprit: an object/array/function recreated every render.
```jsx
// ❌ options is new every render → effect runs forever
const options = { id };
useEffect(() => { fetchData(options); }, [options]);

// ✅ Depend on primitives, or memoize the object
useEffect(() => { fetchData({ id }); }, [id]);
```

### S4. "I fetch data in useEffect, but I get a warning about setting state on an unmounted component. How do I fix it?"
The component unmounted before the async request finished. Use an **ignore flag** or `AbortController` in cleanup.
```jsx
useEffect(() => {
  let active = true;
  fetch(url).then(r => r.json()).then(data => {
    if (active) setData(data);
  });
  return () => { active = false; };
}, [url]);
```

### S5. "Typing in my controlled input feels laggy on a big form/list. What can I do?"
Each keystroke re-renders. Options:
- Isolate the input into its own small component so typing doesn't re-render the whole tree.
- Debounce expensive work (filtering/search) with `useDeferredValue` or a debounce.
- Memoize heavy children with `React.memo`.

### S6. "A list re-orders and now the wrong items show wrong state (e.g., checkboxes). Why?"
You're using the **array index as the key**. When items reorder, React reuses DOM/state by position, not identity. Use a stable unique `id` as the key.

### S7. "Clicking a button that uses a value from state shows the OLD value. Why?"
This is a **stale closure**. The handler captured the state from the render when it was created. Use the functional updater, or ensure the effect/handler has correct dependencies.
```jsx
setCount(c => c + 1); // always uses latest
```

### S8. "How would you share the logged-in user across the whole app without prop drilling?"
Use the **Context API**: create an `AuthContext`, wrap the app in its Provider with the user value, and consume it with `useContext` wherever needed. For large/complex state, reach for Redux/Zustand instead.

### S9. "A child component re-renders every time the parent renders, even though its props look the same. Why?"
The parent passes a **new object/array/function reference** each render (e.g., an inline arrow or `{...}`). Wrap the child in `React.memo` and stabilize the props with `useCallback`/`useMemo`.
```jsx
const handleClick = useCallback(() => doThing(id), [id]);
```

### S10. "I need to focus an input as soon as a modal opens. How?"
Use a **ref** plus an effect:
```jsx
const inputRef = useRef(null);
useEffect(() => { inputRef.current?.focus(); }, []);
<input ref={inputRef} />
```

### S11. "I want to reset a form component entirely when the selected user changes. Easiest trick?"
Change the component's **`key`**. A new key makes React unmount and remount it, resetting all internal state.
```jsx
<UserForm key={selectedUserId} user={user} />
```

### S12. "My app crashes completely when one widget throws an error. How do I contain it?"
Wrap that part of the tree in an **Error Boundary** so only that section shows a fallback UI while the rest of the app keeps working.

### S13. "How would you fetch data only when a button is clicked, not on mount?"
Don't put it in `useEffect`. Call the fetch inside the click handler and store the result in state.
```jsx
const loadData = async () => {
  const res = await fetch(url);
  setData(await res.json());
};
<button onClick={loadData}>Load</button>
```

### S14. "Two sibling components need to stay in sync. How do you structure it?"
**Lift the state up** to their nearest common parent. The parent holds the state and passes it down (plus setters) to both children, giving a single source of truth.

### S15. "My initial page shows a flash/flicker before an effect adjusts the layout. Which hook helps?"
`useLayoutEffect` — it runs synchronously **before the browser paints**, so DOM measurements/adjustments happen before the user sees anything. (Use sparingly; prefer `useEffect` otherwise.)

### S16. "How do you handle a very long list of 10,000 rows without freezing the browser?"
Use **virtualization/windowing** (e.g., `react-window`) to render only the rows currently visible, plus a small buffer, instead of all 10,000 DOM nodes.

### S17. "You need to debounce a search API call as the user types. How?"
Debounce inside an effect keyed on the query, and clear the timer on cleanup.
```jsx
useEffect(() => {
  const id = setTimeout(() => search(query), 400);
  return () => clearTimeout(id);
}, [query]);
```

### S18. "A useMemo you added didn't improve performance. What might be the issue?"
Either the computation was cheap to begin with (memo overhead isn't worth it), or the dependencies change every render (so it recomputes anyway), or the real bottleneck is elsewhere. **Profile first**, then optimize.

## Next.js Scenarios

### S19. "My page needs great SEO and the content rarely changes. Which rendering strategy?"
**SSG (Static Site Generation)** — pre-render at build time, serve from CDN. Add **ISR** (`revalidate`) if the content updates occasionally without needing a full rebuild.

### S20. "My dashboard shows different data per logged-in user and must always be fresh. Which strategy?"
**SSR** (`getServerSideProps` in Pages Router, or `fetch` with `cache: 'no-store'` in App Router) — rendered per request with the user's data.

### S21. "I get a 'hydration mismatch' error. What causes it and how to fix?"
The server-rendered HTML differs from the first client render — often from using `Date.now()`, `Math.random()`, `window`, or `localStorage` during render. Move that logic into `useEffect` (client-only), or guard with a mounted check.

### S22. "I used useState in an App Router component and got an error. Why?"
App Router components are **Server Components by default**, which can't use state/effects/handlers. Add `'use client'` at the top of the file to make it a Client Component.

### S23. "Where should I put my database query — will it leak to the browser?"
Do it in a **Server Component**, a **Route Handler** (`route.js`), or a **Server Action**. Server code (and secrets like DB URLs) never ships to the client, so it's safe there — never in a Client Component.

### S24. "I want a shared navbar that doesn't reload/re-render when navigating between pages. How?"
Put it in a **`layout.js`**. Layouts persist across navigation within their segment, so the navbar stays mounted while only the page content changes.

### S25. "My API key is showing up in the browser bundle. What went wrong?"
You either prefixed it with `NEXT_PUBLIC_` (which exposes it to the client) or used it inside a Client Component. Keep secrets unprefixed and use them only in server-side code (Route Handlers, Server Components, Server Actions).

### S26. "I need to redirect unauthenticated users away from /dashboard before the page loads. Best place?"
Use **middleware** (`middleware.js`). It runs before the request completes, so you can check auth cookies and redirect at the edge without rendering the page.

### S27. "How do I show a loading spinner while an App Router page fetches data?"
Add a **`loading.js`** file in that route segment. Next.js automatically wraps the page in Suspense and shows it while the server component streams. For errors, add **`error.js`**.

### S28. "I want to submit a form and write to the database without building a separate API route. How?"
Use a **Server Action** (`'use server'`). Attach it to the form's `action`; it runs on the server, performs the mutation, and can revalidate the cache — no manual endpoint needed.

### S29. "Images load slowly and cause layout shift. What's the Next.js fix?"
Use the **`next/image`** `<Image>` component with `width`/`height` (or `fill`). It lazy-loads, serves optimized formats (WebP/AVIF), and reserves space to prevent layout shift.

### S30. "A heavy chart library bloats my bundle and only runs client-side. How do I load it lazily?"
Use **`next/dynamic`** with `ssr: false` so it's split out and loaded only in the browser when needed.
```js
const Chart = dynamic(() => import('./Chart'), { ssr: false });
```

### S31. "Blog posts should be static but update within a minute of editing. Which feature?"
**ISR** — statically generate the pages and set `revalidate: 60`. Pages serve instantly from cache and regenerate in the background after the interval.

### S32. "How would you build a dynamic route for /product/[id] and pre-render popular products?"
Create `app/product/[id]/page.js` (or `pages/product/[id].js`). In the Pages Router, use `getStaticPaths` to list which IDs to pre-render and `fallback` for the rest. In the App Router, use `generateStaticParams`.

### S33. "Client-side navigation feels like a full reload. What am I doing wrong?"
You're probably using a plain `<a href>` instead of Next's **`<Link>`** component (or `router.push`). `<Link>` does client-side navigation without a full page reload.

---

# TRADE-OFF QUESTIONS

These test judgment: there's no single "right" answer — you must weigh pros and cons and say **when** you'd pick each. Good answers name the cost, not just the benefit.

## React Trade-offs

### T1. Local state (useState) vs Global state (Redux/Zustand/Context)
| | Local state | Global state |
|---|---|---|
| **Pros** | Simple, colocated, no boilerplate | Shared anywhere, predictable, devtools |
| **Cons** | Can't share across distant components | Boilerplate, indirection, over-engineering risk |

**How I'd answer:** I keep state local by default and only lift it to global when several distant components genuinely need it — the cost of going global too early is boilerplate and indirection I don't need. So I'd pick **local state** for the vast majority of cases, and **global state** only when prop drilling gets painful or the data is truly app-wide (auth, cart, theme).

### T2. Context API vs Redux
- **Context**: zero dependencies, great for low-frequency data (theme, locale, user). **Cost**: every consumer re-renders when the value changes; no built-in devtools/middleware.
- **Redux**: fine-grained updates, middleware, time-travel debugging, structure for large teams. **Cost**: boilerplate and a learning curve.

**How I'd answer:** I'd reach for **Context** for simple, rarely-changing values like theme or locale — accepting that its cost is re-rendering every consumer when the value changes. I'd pick **Redux** (or Zustand for less boilerplate) once state is large, frequently updated, and complex enough that I need fine-grained updates and devtools, and I'm willing to pay the boilerplate cost for that structure.

### T3. useState vs useReducer
- **useState**: minimal, perfect for independent simple values. **Cost**: gets messy when many values update together or transitions are complex.
- **useReducer**: centralizes complex transition logic, easier to test, predictable. **Cost**: more upfront code, overkill for a boolean.

**How I'd answer:** I'd use **useState** for simple, independent values — reaching for `useReducer` there would just be extra code for no gain. I'd pick **useReducer** when the next state depends on the previous one or several fields change together, accepting the upfront boilerplate cost in exchange for centralized, testable transition logic.

### T4. Controlled vs Uncontrolled components
- **Controlled**: React is the source of truth — easy validation, conditional logic, instant access to values. **Cost**: a re-render per keystroke, more code.
- **Uncontrolled** (refs): less code, better raw performance, closer to native. **Cost**: harder validation and dynamic behavior.

**How I'd answer:** I'd pick **controlled** for interactive forms that need validation or conditional logic, accepting the cost of a re-render per keystroke. I'd go **uncontrolled** (or React Hook Form, which is uncontrolled under the hood) for simple or very large, performance-sensitive forms — trading some validation convenience for fewer re-renders and less code.

### T5. Optimizing with memo/useMemo/useCallback vs leaving it alone
- **Optimizing**: skips wasted renders/recalculations for expensive work.
- **Cost**: memoization has its own overhead (memory + comparisons), adds complexity, and often no measurable gain. Wrong dependency arrays cause subtle bugs.

**How I'd answer:** I don't memoize by default — the cost is memory, comparison overhead, and complexity that usually buys nothing. So I'd **leave it alone** and profile first with React DevTools, then apply **memo/useMemo/useCallback** only to genuinely expensive components/computations or to stabilize props for a memoized child. I treat premature memoization as an anti-pattern.

### T6. Composition (children/props) vs Inheritance / HOCs
- **Composition & hooks**: flexible, explicit, the modern React way.
- **HOCs**: reusable cross-cutting logic. **Cost**: wrapper hell, prop-name collisions, unclear data flow.

**How I'd answer:** I'd default to **composition and custom hooks** — they're explicit and avoid the wrapper hell and prop collisions that HOCs bring. I'd only reach for **HOCs** in legacy codebases or when wrapping third-party components, accepting their indirection cost because there's no cleaner option there.

### T7. Many small components vs Few large components
- **Small**: reusable, testable, readable, easier to memoize. **Cost**: more files, prop-passing overhead, potential over-abstraction.
- **Large**: fewer files, less indirection. **Cost**: hard to reuse/test, re-renders more, harder to read.

**How I'd answer:** I split by **responsibility**, not line count. I'd extract a **smaller component** when it's reused, independently testable, or the parent has grown hard to follow — but I won't over-split just to hit a size target, since the cost of that is needless files and prop-passing overhead. So I'd keep it **larger** until one of those real triggers appears.

### T8. Client-side data fetching (useEffect) vs a data library (React Query/SWR)
- **useEffect fetch**: no dependency, full control. **Cost**: you hand-roll caching, loading/error states, retries, dedup, and stale data.
- **React Query/SWR**: caching, background refetch, dedup, retries for free.  **Cost**: extra dependency, learning curve.

**How I'd answer:** I'd hand-roll a **useEffect fetch** for a one-off simple request where a library isn't worth the dependency. But I'd pick **React Query/SWR** as soon as I have multiple endpoints, caching needs, or shared server state — accepting the extra dependency because it removes a whole class of caching, retry, and stale-data bugs I'd otherwise write by hand.

### T9. CSS-in-JS vs CSS Modules vs Tailwind
- **CSS-in-JS** (styled-components/emotion): dynamic, scoped, colocated. **Cost**: runtime overhead, larger bundle, SSR complexity.
- **CSS Modules**: scoped, zero runtime, familiar CSS. **Cost**: less dynamic, more files.
- **Tailwind**: fast to build, consistent, no naming. **Cost**: verbose markup, learning the utility names.

**How I'd answer:** For performance-sensitive apps I'd pick **Tailwind or CSS Modules** since they add no runtime cost — accepting verbose markup or extra files as the trade. I'd choose **CSS-in-JS** only when styling is highly dynamic and colocation matters more to me than the runtime and bundle-size cost it brings.

## Next.js Trade-offs

### T10. SSG vs SSR vs ISR vs CSR
| Strategy | Pros | Cons | Best for |
|---|---|---|---|
| **SSG** | Fastest (CDN), cheap, great SEO | Data stale until rebuild | Blogs, docs, marketing |
| **SSR** | Always fresh, SEO, per-user | Slower TTFB, server cost each request | Dashboards, personalized pages |
| **ISR** | Static speed + periodic freshness | Can serve slightly stale content | News, catalogs, semi-dynamic |
| **CSR** | Simple, interactive, cheap server | Poor SEO, slower first paint | Private admin panels, SPAs |

**How I'd answer:** I default to **static (SSG/ISR)** for public content because it's fastest and cheapest — the cost being some data staleness, which ISR's revalidate window keeps acceptable. I'd pick **SSR** only when data must be fresh per request or personalized, accepting the slower TTFB and per-request server cost. And I'd use **CSR** for behind-login interactive dashboards where SEO doesn't matter and a cheap server is a plus.

### T11. Next.js vs plain React (Vite/CRA)
- **Next.js**: routing, SSR/SSG, API routes, image/font optimization, SEO — batteries included. **Cost**: opinionated structure, server infrastructure, more concepts to learn.
- **Plain React**: lightweight, total freedom, simplest for pure SPAs. **Cost**: you assemble routing, SSR, optimization yourself; weak SEO.

**How I'd answer:** I'd pick **Next.js** for content sites, SEO-critical apps, or full-stack needs, accepting its opinionated structure and server infrastructure as the cost of getting routing, rendering, and optimization for free. I'd choose **plain React with Vite** for internal tools, dashboards, or pure client-side SPAs — trading away SSR/SEO for a lighter, freer setup.

### T12. Server Components vs Client Components
- **Server Components**: no JS shipped (smaller bundle), direct data/DB access, keep secrets server-side. **Cost**: no state/effects/interactivity, can't use browser APIs.
- **Client Components**: full interactivity and hooks. **Cost**: ship JS, can't touch the DB/secrets directly.

**How I'd answer:** I'd make everything a **Server Component** by default to keep JS off the client and data access secure — its cost being no interactivity. Then I'd **push `'use client'` down to the leaves** that genuinely need state or events (buttons, inputs, widgets), paying the JS-bundle cost only there. That way the interactive cost stays local and the bundle stays small.

### T13. App Router vs Pages Router
- **App Router**: Server Components, layouts, streaming, Server Actions — the future. **Cost:** newer, some libraries lag, steeper mental model.
- **Pages Router**: mature, stable, huge ecosystem, simpler `getX` data functions. **Cost:** larger client bundles, no Server Components.

**How I'd answer:** For a new project I'd pick the **App Router** — it's what the Next.js team recommends and gives me Server Components and streaming, and I accept the steeper mental model as the cost. I'd stay on the **Pages Router** for existing apps or when a critical dependency isn't App-Router-ready yet, trading the newer features for maturity and stability.

### T14. Server Actions vs API Routes
- **Server Actions**: call server logic directly from components/forms, less boilerplate, progressive enhancement. **Cost:** newer, less suited to public/third-party APIs.
- **API Routes / Route Handlers**: standard REST endpoints usable by any client (mobile, external). **Cost:** more wiring (fetch, JSON, status codes).

**How I'd answer:** I'd pick **Server Actions** for internal form submissions and mutations inside my app, since they cut the boilerplate — accepting that they're newer and less suited to public consumption. I'd use **API Routes/Route Handlers** when I need a standard endpoint that external clients, mobile apps, or webhooks can call, paying the extra wiring cost for that reach.

### T15. next/image vs a plain `<img>`
- **next/image**: auto optimization, lazy loading, no layout shift, modern formats.
- **Cost:** requires width/height config, needs the optimization server (or a loader), remote domains must be allow-listed, occasional friction with certain layouts.

**How I'd answer:** I'd use **next/image** for almost all content images — accepting the config cost of width/height and allow-listing remote domains in exchange for automatic optimization and no layout shift. I'd drop to a plain **`<img>`** for tiny static icons, SVGs, or cases where I deliberately want to opt out of optimization.

### T16. Deploying on Vercel vs self-hosting
- **Vercel**: zero-config, edge network, ISR/preview deploys just work. **Cost:** vendor lock-in feel, pricing at scale.
- **Self-host** (Node/Docker): full control, cost predictability. **Cost:** you manage scaling, caching, edge, and CI yourself.

**How I'd answer:** I'd pick **Vercel** for speed-to-ship and first-class Next.js features like ISR and preview deploys, accepting some vendor lock-in and scale pricing as the cost. I'd **self-host** on Node/Docker when I have specific infra or compliance requirements or cost constraints at scale — trading the zero-config convenience for full control.

---

*Tip: For interviews, be ready to explain the **why** behind each concept and give a small code example. Understanding trade-offs (when NOT to use something) impresses more than memorizing definitions. For scenario questions, state the root cause first, then the fix. For trade-off questions, always name the **cost** of your choice, not just its benefit — and end with a clear "when I'd pick each."*
