# React & Frontend Frameworks — Interview Q&A

---

## 1. What is React and why is it popular?

React is a **JavaScript library for building user interfaces**, created by Meta (Facebook).

**Why popular:**
- **Component-based:** Reusable UI building blocks
- **Virtual DOM:** Efficient updates by diffing virtual DOM vs real DOM
- **Declarative:** Describe what UI should look like, React handles how
- **Large ecosystem:** Next.js, React Native, massive community
- **One-way data flow:** Predictable state management

---

## 2. What is the Virtual DOM? How does it improve performance?

The **Virtual DOM** is an in-memory representation of the real DOM.

**Process (Reconciliation):**
1. State changes → React creates a new Virtual DOM tree
2. React **diffs** the new tree against the previous one
3. Calculates the **minimum set of changes** needed
4. **Batches** and applies only those changes to the real DOM

**Why faster:** Direct DOM manipulation is expensive (causes reflow/repaint). Batching changes minimizes DOM operations.

---

## 3. What are React Hooks? Explain the most common ones.

Hooks let you use state and lifecycle features in **functional components**.

| Hook | Purpose |
|------|---------|
| `useState` | Local state in a component |
| `useEffect` | Side effects (API calls, subscriptions, DOM updates) |
| `useContext` | Access context without prop drilling |
| `useRef` | Persist a mutable value across renders (doesn't trigger re-render) |
| `useMemo` | Memoize expensive computations |
| `useCallback` | Memoize functions (prevent unnecessary re-renders of children) |
| `useReducer` | Complex state logic (like Redux, but local) |

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    document.title = `Count: ${count}`;
  }, [count]); // Only runs when count changes

  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

---

## 4. What is the difference between `useEffect` with different dependency arrays?

```jsx
useEffect(() => { ... });          // Runs after EVERY render
useEffect(() => { ... }, []);      // Runs ONCE after initial render (componentDidMount)
useEffect(() => { ... }, [dep]);   // Runs when `dep` changes
useEffect(() => {
  return () => { cleanup(); };     // Cleanup function (componentWillUnmount)
}, []);
```

---

## 5. What is the difference between state and props?

| Feature | State | Props |
|---------|-------|-------|
| Owned by | The component itself | Parent component |
| Mutable | Yes (via setState/useState) | No (read-only) |
| Triggers re-render | Yes | Yes (when parent re-renders) |
| Purpose | Internal component data | Pass data between components |

**Rule:** Data flows down (parent → child via props). Events flow up (child → parent via callback props).

---

## 6. What is Context API and when should you use it?

Context provides a way to pass data through the component tree **without prop drilling**.

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
  return <ThemedButton />; // No need to pass theme prop
}

function ThemedButton() {
  const theme = useContext(ThemeContext); // "dark"
  return <button className={theme}>Click</button>;
}
```

**Use for:** Theme, auth state, locale, user preferences. **Don't** use for frequently changing state (causes all consumers to re-render).

---

## 7. Explain React component lifecycle (functional components).

```
Mount:     useState initializer → render → DOM update → useEffect([], ...)
Update:    State/prop change → render → DOM update → useEffect([dep], ...)
Unmount:   useEffect cleanup function runs
```

**Class component equivalents:**
- `componentDidMount` → `useEffect(() => {}, [])`
- `componentDidUpdate` → `useEffect(() => {}, [dep])`
- `componentWillUnmount` → `useEffect(() => { return () => cleanup() }, [])`

---

## 8. What is the difference between controlled and uncontrolled components?

**Controlled:** React state drives the form value.
```jsx
const [value, setValue] = useState('');
<input value={value} onChange={e => setValue(e.target.value)} />
```

**Uncontrolled:** DOM drives the value, accessed via ref.
```jsx
const inputRef = useRef();
<input ref={inputRef} />
// Access: inputRef.current.value
```

**Prefer controlled** for most cases (single source of truth, validation, conditional disabling).

---

## 9. What is React.memo, useMemo, and useCallback?

| Tool | What it memoizes | Use case |
|------|-----------------|----------|
| `React.memo` | Component output | Skip re-render if props unchanged |
| `useMemo` | Computed value | Expensive calculations |
| `useCallback` | Function reference | Prevent child re-renders due to new function refs |

```jsx
const MemoizedChild = React.memo(({ onClick }) => <button onClick={onClick}>Click</button>);

function Parent() {
  const [count, setCount] = useState(0);
  const expensive = useMemo(() => computeHeavy(count), [count]);
  const handleClick = useCallback(() => doSomething(), []);
  
  return <MemoizedChild onClick={handleClick} />;
}
```

---

## 10. What is Next.js and why use it over plain React?

Next.js is a **React framework** that adds:

| Feature | Description |
|---------|-------------|
| **Server-Side Rendering (SSR)** | HTML generated on server per request → better SEO, faster first paint |
| **Static Site Generation (SSG)** | HTML generated at build time → fastest, cached on CDN |
| **API Routes** | Backend endpoints in the same project |
| **File-based routing** | `pages/about.tsx` → `/about` route |
| **Image optimization** | Automatic lazy loading, resizing |
| **App Router (v13+)** | React Server Components, layouts, streaming |

**Use Next.js when:** SEO matters, you need SSR/SSG, you want full-stack in one project.
**Use plain React when:** SPA is fine (dashboards, internal tools), no SEO needed.

---

## 11. What are Server Components vs Client Components in Next.js 13+?

| Feature | Server Component | Client Component |
|---------|-----------------|------------------|
| Rendered | On server only | On client (hydrated) |
| Bundle size | Zero JS sent to client | Included in JS bundle |
| Can use | async/await, DB queries, fs | useState, useEffect, event handlers |
| Directive | Default (no directive) | `"use client"` at top |

**Rule of thumb:** Keep components server-side by default. Add `"use client"` only when you need interactivity.

---

## 12. What is client-side rendering (CSR) vs server-side rendering (SSR) vs static site generation (SSG)?

| | CSR | SSR | SSG |
|---|---|---|---|
| HTML generated | In browser (JS) | On server (per request) | At build time |
| First paint | Slow (download + parse JS) | Fast (HTML ready) | Fastest (pre-built) |
| SEO | Poor (empty HTML initially) | Good | Best |
| Server load | None | High (renders every request) | None (served from CDN) |
| Data freshness | Real-time | Real-time | Stale until rebuild |
| Use case | Dashboards, SPAs | E-commerce, news | Blogs, docs, marketing |

---

## 13. How does React handle keys in lists?

Keys help React **identify which items changed, were added, or removed** during reconciliation.

```jsx
// Bad — index as key (causes bugs on reorder/delete)
items.map((item, index) => <li key={index}>{item.name}</li>)

// Good — stable unique ID
items.map(item => <li key={item.id}>{item.name}</li>)
```

**Without proper keys:** React may reuse DOM elements incorrectly → input values persist in wrong items, animations break.

---

## 14. What is code splitting and lazy loading in React?

```jsx
const HeavyComponent = React.lazy(() => import('./HeavyComponent'));

function App() {
  return (
    <Suspense fallback={<Spinner />}>
      <HeavyComponent />
    </Suspense>
  );
}
```

**Benefits:** Only loads code when needed → smaller initial bundle → faster page load.

In Next.js, use `dynamic`:
```jsx
const Chart = dynamic(() => import('./Chart'), { ssr: false });
```

---

## 15. What is state management in React? Compare approaches.

| Approach | Scope | Best For |
|----------|-------|----------|
| `useState` | Component-level | Simple local state |
| `useReducer` | Component-level | Complex state logic |
| Context API | Subtree | Theme, auth, locale |
| Redux / Zustand | Global | Large apps, complex shared state |
| React Query / SWR | Server state | API data caching, sync |

**Modern trend:** Use React Query/SWR for server state, Zustand for client state, avoid Redux unless needed.
