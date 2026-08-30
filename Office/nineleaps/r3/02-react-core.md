# React Core — Round 3 Prep (Uber)

---

## 1. Functional vs Class Components

**Class Component:**
```javascript
class Counter extends React.Component {
    constructor(props) {
        super(props);
        this.state = { count: 0 };
    }
    increment() { this.setState({ count: this.state.count + 1 }); }
    render() {
        return <button onClick={() => this.increment()}>{this.state.count}</button>;
    }
}
```

**Functional Component:**
```javascript
function Counter() {
    const [count, setCount] = useState(0);
    return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

**Why functional components won:**
- Less boilerplate
- Hooks give full power (state, lifecycle, context)
- Easier to test and reason about
- Better performance (no `this` binding issues)

---

## 2. React Lifecycle

**Class lifecycle methods → Hook equivalents:**

| Lifecycle | Hook Equivalent |
|---|---|
| `componentDidMount` | `useEffect(() => {}, [])` |
| `componentDidUpdate` | `useEffect(() => {}, [dep])` |
| `componentWillUnmount` | `useEffect(() => { return () => cleanup }, [])` |
| `shouldComponentUpdate` | `React.memo` / `useMemo` |

```javascript
useEffect(() => {
    // componentDidMount — runs once on mount
    fetchData();

    return () => {
        // componentWillUnmount — cleanup on unmount
        subscription.unsubscribe();
    };
}, []); // empty array = run only on mount
```

---

## 3. Hooks

### useState
```javascript
const [state, setState] = useState(initialValue);

// Functional update — use when new state depends on old state
setCount(prev => prev + 1);
```

### useEffect
```javascript
// Run on every render
useEffect(() => { ... });

// Run only on mount
useEffect(() => { ... }, []);

// Run when dep changes
useEffect(() => { ... }, [dep]);

// Cleanup
useEffect(() => {
    const timer = setInterval(() => {}, 1000);
    return () => clearInterval(timer); // cleanup
}, []);
```

### useLayoutEffect
- Runs **synchronously** after DOM mutations but **before** browser paint
- Use when you need to measure DOM or prevent visual flicker
- `useEffect` runs async after paint — use for most cases
- `useLayoutEffect` runs sync before paint — use for DOM measurements

```javascript
useLayoutEffect(() => {
    // Measure DOM element before user sees it
    const height = ref.current.getBoundingClientRect().height;
    setHeight(height);
}, []);
```

### useRef
Two main uses:

**1. Access DOM element:**
```javascript
const inputRef = useRef(null);
<input ref={inputRef} />
inputRef.current.focus(); // direct DOM access
```

**2. Persist value without re-render:**
```javascript
const countRef = useRef(0);
countRef.current++; // changing this does NOT trigger re-render
```

---

## 4. useMemo vs useCallback

### useMemo — memoizes a **value**
```javascript
const expensiveResult = useMemo(() => {
    return heavyComputation(data); // only recomputes when data changes
}, [data]);
```

### useCallback — memoizes a **function**
```javascript
const handleClick = useCallback(() => {
    doSomething(id);
}, [id]); // only creates new function when id changes
```

**When to use:**
- `useMemo` — expensive calculations, filtered/sorted lists
- `useCallback` — functions passed as props to memoized child components

**Rule:** Don't overuse. Memoization has its own cost. Only use when you have a proven performance issue.

---

## 5. React.memo
Prevents re-render if props haven't changed.

```javascript
const Child = React.memo(function Child({ name }) {
    console.log("Child rendered");
    return <div>{name}</div>;
});

// Child only re-renders if 'name' prop changes
```

**Common mistake:** React.memo does shallow comparison. If you pass a new object/function on every render, memo won't help.

```javascript
// Wrong — new function on every render, memo useless
<Child onClick={() => doSomething()} />

// Fix — wrap with useCallback
const handleClick = useCallback(() => doSomething(), []);
<Child onClick={handleClick} />
```

---

## 6. Context API

Avoid prop drilling by sharing state across the component tree.

```javascript
// 1. Create context
const ThemeContext = createContext("light");

// 2. Provide value
function App() {
    const [theme, setTheme] = useState("light");
    return (
        <ThemeContext.Provider value={{ theme, setTheme }}>
            <Child />
        </ThemeContext.Provider>
    );
}

// 3. Consume anywhere in tree
function DeepChild() {
    const { theme, setTheme } = useContext(ThemeContext);
    return <button onClick={() => setTheme("dark")}>{theme}</button>;
}
```

**Limitation:** Every consumer re-renders when context value changes, even if they don't use the changed part. For large apps, use Redux or split contexts.

---

## 7. Props Drilling & How to Avoid It

**Props drilling** — passing props through multiple layers of components that don't need them, just to reach a deeply nested component.

```
App (has user)
 └── Layout (passes user)
      └── Sidebar (passes user)
           └── UserAvatar (finally uses user) ← only this needs it
```

**Solutions:**
1. **Context API** — for global state (theme, auth, language)
2. **Redux** — for complex global state
3. **Component composition** — pass components as children instead of data
4. **Custom hooks** — encapsulate data fetching closer to where it's needed

---

## 8. Virtual DOM & How React Works

**Virtual DOM** — a lightweight JavaScript representation of the real DOM kept in memory.

**How React updates the UI:**
1. State/props change → React creates new Virtual DOM tree
2. React **diffs** new tree vs old tree (reconciliation)
3. React calculates minimum changes needed
4. React updates only the changed parts of the real DOM (commit phase)

This is faster than directly manipulating the DOM every time because DOM operations are expensive.

**Reconciliation rules:**
- Different element type → destroy and rebuild subtree
- Same element type → update attributes, recurse on children
- Lists → use `key` prop to identify which items changed

**Why keys matter:**
```javascript
// Without keys — React re-renders entire list on any change
{items.map(item => <Item name={item.name} />)}

// With keys — React knows exactly which item changed
{items.map(item => <Item key={item.id} name={item.name} />)}
```

---

## 9. Batching in React

**Batching** — React groups multiple state updates into a single re-render for performance.

**React 17 — only batched inside event handlers:**
```javascript
// React 17: batched — one re-render
function handleClick() {
    setCount(c => c + 1);
    setName("Alice");
}

// React 17: NOT batched — two re-renders
setTimeout(() => {
    setCount(c => c + 1); // re-render
    setName("Alice");     // re-render
}, 0);
```

**React 18 — automatic batching everywhere:**
```javascript
// React 18: batched in setTimeout, Promises, event handlers — one re-render
setTimeout(() => {
    setCount(c => c + 1);
    setName("Alice");     // only one re-render
}, 0);
```

**Opt out of batching (rare):**
```javascript
import { flushSync } from "react-dom";
flushSync(() => setCount(c => c + 1)); // forces immediate re-render
```

---

## 10. Suspense

Lets you show a fallback UI while a component is loading.

```javascript
const LazyComponent = React.lazy(() => import("./HeavyComponent"));

function App() {
    return (
        <Suspense fallback={<div>Loading...</div>}>
            <LazyComponent />
        </Suspense>
    );
}
```

**With data fetching (React 18 + TanStack Query):**
```javascript
function UserProfile() {
    const { data } = useSuspenseQuery({ queryKey: ["user"], queryFn: fetchUser });
    return <div>{data.name}</div>; // no loading state needed — Suspense handles it
}

<Suspense fallback={<Spinner />}>
    <UserProfile />
</Suspense>
```

---

## 11. Concurrent Features / React 18

React 18 introduced **concurrent rendering** — React can pause, resume, and abandon renders.

Key features:

**`startTransition`** — mark updates as non-urgent. UI stays responsive.
```javascript
import { startTransition } from "react";

function handleSearch(query) {
    setInputValue(query);          // urgent — update input immediately
    startTransition(() => {
        setSearchResults(query);   // non-urgent — can be interrupted
    });
}
```

**`useDeferredValue`** — defer updating a value until urgent updates are done.
```javascript
function SearchResults({ query }) {
    const deferredQuery = useDeferredValue(query);
    // deferredQuery updates after urgent updates
    // shows stale results while new ones load
    const results = useMemo(() => filterData(deferredQuery), [deferredQuery]);
    return <List items={results} />;
}
```

**When to use:**
- `startTransition` — for triggered updates (button click, search)
- `useDeferredValue` — for received values (props from parent)

---

## 12. Routing & Data Router

**Basic React Router v6:**
```javascript
import { BrowserRouter, Routes, Route, Link, useNavigate, useParams } from "react-router-dom";

function App() {
    return (
        <BrowserRouter>
            <Routes>
                <Route path="/" element={<Home />} />
                <Route path="/movies/:id" element={<MovieDetail />} />
                <Route path="*" element={<NotFound />} />
            </Routes>
        </BrowserRouter>
    );
}

function MovieDetail() {
    const { id } = useParams();
    const navigate = useNavigate();
    return <button onClick={() => navigate(-1)}>Back</button>;
}
```

**Data Router (v6.4+) — load data before rendering:**
```javascript
const router = createBrowserRouter([
    {
        path: "/movies/:id",
        element: <MovieDetail />,
        loader: async ({ params }) => {
            return fetch(`/api/movies/${params.id}`);
        },
        errorElement: <ErrorPage />
    }
]);

// In component:
function MovieDetail() {
    const movie = useLoaderData();
    return <div>{movie.title}</div>;
}
```

**Benefits of Data Router:**
- Data fetching happens in parallel with rendering
- No waterfall (component loads and then fetches)
- Built-in error and loading states
