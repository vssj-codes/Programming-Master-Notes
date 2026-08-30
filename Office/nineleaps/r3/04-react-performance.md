# React Performance & Optimization — Round 3 Prep (Uber)

---

## 1. Rendering Large Datasets (50K–90K Records)

Rendering 90K rows directly in the DOM will crash the browser. The solution is **virtualization** — only render what's visible on screen.

### React Virtual / TanStack Virtual
```javascript
import { useVirtualizer } from "@tanstack/react-virtual";

function VirtualList({ items }) {
    const parentRef = useRef(null);

    const virtualizer = useVirtualizer({
        count: items.length,
        getScrollElement: () => parentRef.current,
        estimateSize: () => 50, // estimated row height in px
    });

    return (
        <div ref={parentRef} style={{ height: "600px", overflow: "auto" }}>
            <div style={{ height: `${virtualizer.getTotalSize()}px`, position: "relative" }}>
                {virtualizer.getVirtualItems().map(virtualItem => (
                    <div
                        key={virtualItem.key}
                        style={{
                            position: "absolute",
                            top: `${virtualItem.start}px`,
                            height: `${virtualItem.size}px`,
                        }}
                    >
                        {items[virtualItem.index].name}
                    </div>
                ))}
            </div>
        </div>
    );
}
```

**How it works:**
- Only renders ~20 rows visible on screen
- As user scrolls, removes rows leaving viewport, adds rows entering viewport
- DOM always has ~20 nodes regardless of total data size

**Alternative:** `react-window` or `react-virtualized`

---

## 2. Search & Filtering Optimization

**Problem:** Filtering 90K records on every keystroke = UI freeze.

**Solution 1: Debounce the search**
```javascript
const [query, setQuery] = useState("");
const [results, setResults] = useState(data);

const debouncedSearch = useCallback(
    debounce((q) => {
        setResults(data.filter(item =>
            item.name.toLowerCase().includes(q.toLowerCase())
        ));
    }, 300),
    [data]
);

const handleChange = (e) => {
    setQuery(e.target.value);
    debouncedSearch(e.target.value);
};
```

**Solution 2: useDeferredValue (React 18)**
```javascript
function Search({ data }) {
    const [query, setQuery] = useState("");
    const deferredQuery = useDeferredValue(query); // defer expensive filtering

    const results = useMemo(
        () => data.filter(item => item.name.includes(deferredQuery)),
        [data, deferredQuery]
    );

    return (
        <>
            <input value={query} onChange={e => setQuery(e.target.value)} />
            {/* input stays responsive, results update when idle */}
            <VirtualList items={results} />
        </>
    );
}
```

**Solution 3: Web Worker for heavy computation**
```javascript
// Move filtering to a background thread
const worker = new Worker("./searchWorker.js");
worker.postMessage({ data, query });
worker.onmessage = (e) => setResults(e.data);
```

---

## 3. Preventing Unnecessary Re-renders

### When does a component re-render?
1. Its state changes
2. Its props change
3. Its parent re-renders
4. Its context value changes

### Tools to prevent unnecessary re-renders

**React.memo — skip re-render if props unchanged:**
```javascript
const Row = React.memo(({ item }) => {
    return <div>{item.name}</div>;
});
```

**useCallback — stable function reference:**
```javascript
// Without useCallback — new function on every render → Row re-renders
const handleDelete = (id) => deleteItem(id);

// With useCallback — same function reference → Row doesn't re-render
const handleDelete = useCallback((id) => deleteItem(id), []);
```

**useMemo — stable object/array reference:**
```javascript
// Without useMemo — new array on every render
const filtered = items.filter(i => i.active);

// With useMemo — same array reference if items unchanged
const filtered = useMemo(() => items.filter(i => i.active), [items]);
```

**Avoid passing new objects/arrays as props:**
```javascript
// Bad — new object on every render
<Child config={{ size: "large", color: "blue" }} />

// Good — stable reference
const config = useMemo(() => ({ size: "large", color: "blue" }), []);
<Child config={config} />
```

---

## 4. Memoization

Already covered in JS fundamentals. In React context:

```javascript
// Expensive computation — wrap in useMemo
const sortedData = useMemo(() => {
    return [...data].sort((a, b) => b.price - a.price);
}, [data]);

// Expensive component — wrap in React.memo
const ExpensiveChart = React.memo(({ data }) => {
    return <Chart data={data} />;
});
```

**Profile first:** Use React DevTools Profiler to identify what's actually slow before adding memoization.

---

## 5. Debouncing

Already covered in JS fundamentals. Custom React hook:

```javascript
function useDebounce(value, delay) {
    const [debouncedValue, setDebouncedValue] = useState(value);

    useEffect(() => {
        const timer = setTimeout(() => setDebouncedValue(value), delay);
        return () => clearTimeout(timer);
    }, [value, delay]);

    return debouncedValue;
}

// Usage
function Search() {
    const [query, setQuery] = useState("");
    const debouncedQuery = useDebounce(query, 300);

    useEffect(() => {
        if (debouncedQuery) fetch(`/search?q=${debouncedQuery}`);
    }, [debouncedQuery]);

    return <input value={query} onChange={e => setQuery(e.target.value)} />;
}
```

---

## 6. Lazy Loading

**Code splitting — load components only when needed:**
```javascript
import { lazy, Suspense } from "react";

// Component is loaded only when rendered for the first time
const HeavyDashboard = lazy(() => import("./HeavyDashboard"));

function App() {
    return (
        <Suspense fallback={<Spinner />}>
            <HeavyDashboard />
        </Suspense>
    );
}
```

**Route-based lazy loading (most common):**
```javascript
const Home = lazy(() => import("./pages/Home"));
const Dashboard = lazy(() => import("./pages/Dashboard"));
const Reports = lazy(() => import("./pages/Reports"));

function App() {
    return (
        <Suspense fallback={<PageLoader />}>
            <Routes>
                <Route path="/" element={<Home />} />
                <Route path="/dashboard" element={<Dashboard />} />
                <Route path="/reports" element={<Reports />} />
            </Routes>
        </Suspense>
    );
}
```

**Lazy load images:**
```javascript
// Native lazy loading
<img src="photo.jpg" loading="lazy" alt="photo" />

// Or Intersection Observer for custom behavior
```

---

## 7. Preload vs Prefetch

Both are HTML `<link>` hints to the browser. Different priorities.

**Preload** — fetch this resource NOW, it's needed for the current page:
```html
<link rel="preload" href="/fonts/inter.woff2" as="font" crossorigin />
<link rel="preload" href="/api/critical-data" as="fetch" crossorigin />
```
Use for: fonts, critical CSS, hero images, critical API calls.

**Prefetch** — fetch this resource WHEN IDLE, it'll be needed soon (future navigation):
```html
<link rel="prefetch" href="/dashboard.js" as="script" />
```
Use for: next page's JS bundle, resources for likely next navigation.

**In React (webpack):**
```javascript
// Prefetch — low priority, loads when browser is idle
const Dashboard = lazy(() => import(/* webpackPrefetch: true */ "./Dashboard"));

// Preload — high priority, loads immediately
const Modal = lazy(() => import(/* webpackPreload: true */ "./Modal"));
```

| | Preload | Prefetch |
|---|---|---|
| Priority | High — needed now | Low — needed soon |
| Timing | Immediately | When browser is idle |
| Use for | Current page critical resources | Next page resources |

---

## 8. Efficient API/Data Handling

**Avoid fetching in every component — lift data up or use a cache:**
```javascript
// Bad — multiple components fetch same data independently
function Header() { useFetch("/api/user"); }
function Sidebar() { useFetch("/api/user"); } // duplicate request

// Good — fetch once, share via context/store
// Or use TanStack Query — it deduplicates requests automatically
```

**Pagination vs Infinite scroll:**
```javascript
// Pagination — load page N
const { data } = useQuery({
    queryKey: ["users", page],
    queryFn: () => fetch(`/api/users?page=${page}`)
});

// Infinite scroll with TanStack Query
const { data, fetchNextPage } = useInfiniteQuery({
    queryKey: ["users"],
    queryFn: ({ pageParam = 0 }) => fetch(`/api/users?cursor=${pageParam}`),
    getNextPageParam: (lastPage) => lastPage.nextCursor,
});
```

**Cancel stale requests:**
```javascript
useEffect(() => {
    const controller = new AbortController();

    fetch(`/api/search?q=${query}`, { signal: controller.signal })
        .then(res => res.json())
        .then(setResults)
        .catch(err => {
            if (err.name !== "AbortError") setError(err);
        });

    return () => controller.abort(); // cancel on cleanup
}, [query]);
```
