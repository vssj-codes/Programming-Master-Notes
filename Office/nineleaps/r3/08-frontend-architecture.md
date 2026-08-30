# Frontend Architecture & UI — Round 3 Prep (Uber)

---

## 1. Project Structure

**Feature-based structure (recommended for large apps):**
```
src/
  features/
    auth/
      components/
        LoginForm.jsx
        ProtectedRoute.jsx
      hooks/
        useAuth.js
      store/
        authSlice.js
      api/
        authApi.js
      index.js          ← public API of the feature
    movies/
      components/
      hooks/
      store/
      api/
  shared/
    components/         ← reusable UI components
      Button/
      Modal/
      Input/
    hooks/              ← shared custom hooks
    utils/              ← utility functions
    constants/
  pages/                ← route-level components
    Home.jsx
    Dashboard.jsx
  app/
    store.js            ← Redux store
    router.jsx          ← routes config
    App.jsx
  main.jsx
```

**Rule:** components should not import from sibling features. Features communicate through shared store or events.

---

## 2. Creating a New Project from Scratch

```bash
# Vite (recommended — fast, modern)
npm create vite@latest my-app -- --template react
cd my-app
npm install

# Install common dependencies
npm install react-router-dom @tanstack/react-query axios
npm install @reduxjs/toolkit react-redux
npm install -D @testing-library/react @testing-library/jest-dom vitest
```

**Initial setup checklist:**
- [ ] Configure path aliases (`@/` for `src/`)
- [ ] Set up ESLint + Prettier
- [ ] Configure environment variables (`.env`, `.env.production`)
- [ ] Set up base API client (Axios instance)
- [ ] Set up React Router
- [ ] Set up TanStack Query client
- [ ] Set up Redux store (if needed)
- [ ] Set up error boundary
- [ ] Add loading/error states pattern

```javascript
// vite.config.js — path alias
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import path from "path";

export default defineConfig({
    plugins: [react()],
    resolve: {
        alias: { "@": path.resolve(__dirname, "./src") }
    }
});
```

---

## 3. Reusable Component Design

**Principles:**
- Single responsibility — one component, one job
- Controlled via props — no internal side effects
- Composable — works with other components
- Accessible — keyboard navigation, ARIA attributes

**Good reusable Button:**
```javascript
function Button({
    children,
    variant = "primary",    // primary | secondary | danger
    size = "md",            // sm | md | lg
    disabled = false,
    loading = false,
    onClick,
    type = "button",
    ...rest                 // spread remaining props (aria-*, data-*)
}) {
    return (
        <button
            type={type}
            disabled={disabled || loading}
            onClick={onClick}
            className={`btn btn-${variant} btn-${size}`}
            aria-busy={loading}
            {...rest}
        >
            {loading ? <Spinner size="sm" /> : children}
        </button>
    );
}
```

**Compound component pattern (advanced):**
```javascript
// Card with sub-components
function Card({ children, className }) {
    return <div className={`card ${className}`}>{children}</div>;
}
Card.Header = function({ children }) { return <div className="card-header">{children}</div>; };
Card.Body = function({ children }) { return <div className="card-body">{children}</div>; };
Card.Footer = function({ children }) { return <div className="card-footer">{children}</div>; };

// Usage
<Card>
    <Card.Header>Title</Card.Header>
    <Card.Body>Content</Card.Body>
    <Card.Footer>Actions</Card.Footer>
</Card>
```

---

## 4. Component Props/API Design

**Principles:**
- Minimal required props — sensible defaults for optional ones
- Consistent naming — `onX` for callbacks, `isX` for booleans
- Don't leak implementation details through props
- Accept `className` and spread rest props for flexibility

```javascript
// Bad API — too many unrelated props
<Table data={data} onSort={handleSort} sortKey="name"
       showPagination paginationSize={10} currentPage={2}
       onPageChange={handlePage} loading={false} />

// Good API — compose separate concerns
<Table data={data}>
    <Table.Sort onSort={handleSort} defaultKey="name" />
    <Table.Pagination pageSize={10} />
</Table>
```

---

## 5. CSS Frameworks & Libraries

| Library | Style | Best For |
|---|---|---|
| **Tailwind CSS** | Utility classes | Rapid UI, customizable |
| **CSS Modules** | Scoped CSS | No conflicts, any app |
| **styled-components** | CSS-in-JS | Dynamic styles based on props |
| **Material UI (MUI)** | Component library | Enterprise apps |
| **shadcn/ui** | Headless + Tailwind | Modern, customizable components |
| **Ant Design** | Component library | Admin dashboards |

**Tailwind example:**
```jsx
<button className="bg-blue-600 text-white px-4 py-2 rounded hover:bg-blue-700 disabled:opacity-50">
    Submit
</button>
```

**CSS Modules:**
```css
/* Button.module.css */
.button { background: blue; color: white; }
.primary { background: #1971c2; }
```
```javascript
import styles from "./Button.module.css";
<button className={`${styles.button} ${styles.primary}`}>Click</button>
```

---

## 6. RTL/LTR Support

**Right-to-Left (RTL)** — Arabic, Hebrew, Urdu languages read right to left.

```javascript
// Set direction on root element
document.documentElement.dir = isRTL ? "rtl" : "ltr";
document.documentElement.lang = locale; // "ar", "he", "en"
```

**CSS logical properties (auto-adapt to direction):**
```css
/* Instead of left/right — use start/end */
.card {
    margin-inline-start: 16px;  /* = margin-left in LTR, margin-right in RTL */
    padding-inline-end: 16px;   /* = padding-right in LTR, padding-left in RTL */
}
```

**Tailwind RTL:**
```jsx
<div className="text-left rtl:text-right">
    <span className="ml-4 rtl:mr-4 rtl:ml-0">Text</span>
</div>
```

**i18n library (react-i18next):**
```javascript
import i18n from "i18next";
import { initReactI18next } from "react-i18next";

i18n.use(initReactI18next).init({
    lng: "en",
    resources: {
        en: { translation: { greeting: "Hello" } },
        ar: { translation: { greeting: "مرحبا" } }
    }
});

// In component
const { t, i18n } = useTranslation();
<p>{t("greeting")}</p>
<button onClick={() => i18n.changeLanguage("ar")}>Arabic</button>
```

---

## 7. Building Scalable Components with Minimal Props

**Render props pattern:**
```javascript
function DataFetcher({ url, render }) {
    const [data, setData] = useState(null);
    useEffect(() => { fetch(url).then(r => r.json()).then(setData); }, [url]);
    return render(data);
}

<DataFetcher url="/api/users" render={(users) => (
    <ul>{users?.map(u => <li key={u.id}>{u.name}</li>)}</ul>
)} />
```

**Custom hooks for logic separation:**
```javascript
// Extract logic into hook — component stays thin
function useUsers() {
    const [users, setUsers] = useState([]);
    const [loading, setLoading] = useState(true);

    useEffect(() => {
        fetch("/api/users")
            .then(r => r.json())
            .then(data => { setUsers(data); setLoading(false); });
    }, []);

    return { users, loading };
}

// Component only handles rendering
function UserList() {
    const { users, loading } = useUsers();
    if (loading) return <Spinner />;
    return <ul>{users.map(u => <li key={u.id}>{u.name}</li>)}</ul>;
}
```

**Avoid prop drilling with composition:**
```javascript
// Instead of passing user through many layers
function App() {
    const user = useAuth();
    return (
        <Layout>
            <Sidebar user={user} />      // ← drilling
            <Main>
                <Header user={user} />   // ← drilling
            </Main>
        </Layout>
    );
}

// Use children composition
function App() {
    const user = useAuth();
    return (
        <Layout sidebar={<Sidebar />} header={<Header />}>
            <Main />
        </Layout>
        // Sidebar and Header get user from context, not props
    );
}
```
