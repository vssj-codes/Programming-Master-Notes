# Coding Practice — Round 3 Prep (Uber)
# All examples are paste-ready for StackBlitz (stackblitz.com → React)

---

## 1. Debounce Input + Custom Debounce Hook

```javascript
import { useState, useEffect } from "react";

function useDebounce(value, delay = 300) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);

  return debouncedValue;
}

export default function App() {
  const [query, setQuery] = useState("");
  const [results, setResults] = useState([]);
  const debouncedQuery = useDebounce(query, 500);

  useEffect(() => {
    if (!debouncedQuery.trim()) { setResults([]); return; }

    // Using a public API for demo
    fetch(`https://jsonplaceholder.typicode.com/users`)
      .then((r) => r.json())
      .then((data) => {
        const filtered = data.filter((u) =>
          u.name.toLowerCase().includes(debouncedQuery.toLowerCase())
        );
        setResults(filtered);
      });
  }, [debouncedQuery]);

  return (
    <div style={{ padding: 24 }}>
      <h2>Debounce Search</h2>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Type a name..."
        style={{ padding: 8, width: 300 }}
      />
      <ul>
        {results.map((r) => (
          <li key={r.id}>{r.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

---

## 2. Progress Bar / Loader with Concurrent Animations

```javascript
import { useState, useEffect } from "react";

function ProgressBar({ value, label }) {
  return (
    <div style={{ marginBottom: 16 }}>
      <div style={{ marginBottom: 4 }}>{label}: {Math.round(value)}%</div>
      <div style={{ width: "100%", background: "#eee", borderRadius: 4 }}>
        <div
          style={{
            width: `${value}%`,
            height: 16,
            background: "#1971c2",
            borderRadius: 4,
            transition: "width 0.3s ease",
          }}
        />
      </div>
    </div>
  );
}

export default function App() {
  const [progresses, setProgresses] = useState([0, 0, 0]);

  useEffect(() => {
    const intervals = [0, 1, 2].map((i) =>
      setInterval(() => {
        setProgresses((prev) => {
          const next = [...prev];
          if (next[i] < 100) next[i] = Math.min(next[i] + Math.random() * 8, 100);
          return next;
        });
      }, 200 + i * 100)
    );

    return () => intervals.forEach(clearInterval);
  }, []);

  const reset = () => setProgresses([0, 0, 0]);

  return (
    <div style={{ padding: 24, maxWidth: 400 }}>
      <h2>Concurrent Progress Bars</h2>
      {progresses.map((p, i) => (
        <ProgressBar key={i} value={p} label={`Task ${i + 1}`} />
      ))}
      <button onClick={reset} style={{ marginTop: 8, padding: "8px 16px" }}>
        Reset
      </button>
    </div>
  );
}
```

---

## 3. Undo/Redo Functionality

```javascript
import { useState } from "react";

function useUndoRedo(initialState) {
  const [history, setHistory] = useState([initialState]);
  const [index, setIndex] = useState(0);

  const current = history[index];

  const set = (newState) => {
    const newHistory = history.slice(0, index + 1);
    setHistory([...newHistory, newState]);
    setIndex(newHistory.length);
  };

  const undo = () => { if (index > 0) setIndex((i) => i - 1); };
  const redo = () => { if (index < history.length - 1) setIndex((i) => i + 1); };

  return {
    current,
    set,
    undo,
    redo,
    canUndo: index > 0,
    canRedo: index < history.length - 1,
  };
}

export default function App() {
  const { current, set, undo, redo, canUndo, canRedo } = useUndoRedo("");

  return (
    <div style={{ padding: 24 }}>
      <h2>Undo / Redo</h2>
      <div style={{ marginBottom: 8 }}>
        <button onClick={undo} disabled={!canUndo} style={{ marginRight: 8 }}>
          ← Undo
        </button>
        <button onClick={redo} disabled={!canRedo}>
          Redo →
        </button>
      </div>
      <textarea
        value={current}
        onChange={(e) => set(e.target.value)}
        rows={6}
        style={{ width: 400, padding: 8 }}
        placeholder="Type something..."
      />
      <p style={{ color: "#888", fontSize: 13 }}>
        History length: {/* shown implicitly via undo/redo state */}
        {canUndo ? "Can undo" : "Nothing to undo"} |{" "}
        {canRedo ? "Can redo" : "Nothing to redo"}
      </p>
    </div>
  );
}
```

---

## 4. Search Functionality Using API

```javascript
import { useState, useEffect } from "react";

function useDebounce(value, delay = 300) {
  const [debouncedValue, setDebouncedValue] = useState(value);
  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);
  return debouncedValue;
}

export default function App() {
  const [query, setQuery] = useState("");
  const [results, setResults] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const debouncedQuery = useDebounce(query, 400);

  useEffect(() => {
    if (!debouncedQuery.trim()) { setResults([]); return; }

    const controller = new AbortController();
    setLoading(true);
    setError(null);

    fetch(
      `https://jsonplaceholder.typicode.com/users`,
      { signal: controller.signal }
    )
      .then((r) => r.json())
      .then((data) => {
        setResults(
          data.filter((u) =>
            u.name.toLowerCase().includes(debouncedQuery.toLowerCase())
          )
        );
      })
      .catch((err) => {
        if (err.name !== "AbortError") setError("Something went wrong");
      })
      .finally(() => setLoading(false));

    return () => controller.abort();
  }, [debouncedQuery]);

  return (
    <div style={{ padding: 24 }}>
      <h2>Search with API</h2>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search users..."
        style={{ padding: 8, width: 300 }}
      />
      {loading && <p>Loading...</p>}
      {error && <p style={{ color: "red" }}>{error}</p>}
      <ul>
        {results.map((r) => (
          <li key={r.id}>
            <strong>{r.name}</strong> — {r.email}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

---

## 5. Counter with Increment/Decrement

```javascript
import { useState } from "react";

function Counter({ min = 0, max = 10, step = 1 }) {
  const [count, setCount] = useState(0);

  const increment = () => setCount((c) => Math.min(c + step, max));
  const decrement = () => setCount((c) => Math.max(c - step, min));
  const reset = () => setCount(0);

  return (
    <div style={{ display: "flex", alignItems: "center", gap: 12 }}>
      <button onClick={decrement} disabled={count <= min} style={{ fontSize: 20, padding: "4px 12px" }}>
        −
      </button>
      <span style={{ fontSize: 24, minWidth: 40, textAlign: "center" }}>{count}</span>
      <button onClick={increment} disabled={count >= max} style={{ fontSize: 20, padding: "4px 12px" }}>
        +
      </button>
      <button onClick={reset} style={{ padding: "4px 12px" }}>Reset</button>
    </div>
  );
}

export default function App() {
  return (
    <div style={{ padding: 24 }}>
      <h2>Counter</h2>
      <Counter min={0} max={10} step={1} />
    </div>
  );
}
```

---

## 6. JavaScript Utilities

```javascript
export default function App() {

  // Reverse a string
  const reverseString = (str) => str.split("").reverse().join("");

  // Reverse words
  const reverseWords = (str) => str.split(" ").reverse().join(" ");

  // Flatten nested array
  function flatten(arr) {
    return arr.reduce(
      (flat, item) => flat.concat(Array.isArray(item) ? flatten(item) : item),
      []
    );
  }

  // Sum values from nested object
  function sumValues(obj) {
    return Object.values(obj).reduce((sum, val) => {
      if (typeof val === "object" && val !== null) return sum + sumValues(val);
      if (typeof val === "number") return sum + val;
      return sum;
    }, 0);
  }

  // Find key in nested object
  function findValue(obj, targetKey) {
    if (obj[targetKey] !== undefined) return obj[targetKey];
    for (const val of Object.values(obj)) {
      if (typeof val === "object" && val !== null) {
        const found = findValue(val, targetKey);
        if (found !== undefined) return found;
      }
    }
    return undefined;
  }

  // Word occurrence count
  function countWords(str) {
    return str
      .toLowerCase()
      .split(/\s+/)
      .reduce((acc, word) => {
        acc[word] = (acc[word] || 0) + 1;
        return acc;
      }, {});
  }

  return (
    <div style={{ padding: 24, fontFamily: "monospace" }}>
      <h2>JS Utilities</h2>
      <p>reverseString("hello") → {reverseString("hello")}</p>
      <p>reverseWords("Hello World") → {reverseWords("Hello World")}</p>
      <p>flatten([1,[2,[3,[4]]]]) → {JSON.stringify(flatten([1, [2, [3, [4]]]]))}</p>
      <p>sumValues(a:1, b:{"{"}c:2, d:{"{"}e:3{"}"}{"}"}) → {sumValues({ a: 1, b: { c: 2, d: { e: 3 } } })}</p>
      <p>findValue(obj, "c") → {findValue({ a: { b: { c: 42 } } }, "c")}</p>
      <p>countWords("hello world hello") → {JSON.stringify(countWords("hello world hello"))}</p>
    </div>
  );
}
```

---

## 7. Render Nested JSON Recursively

```javascript
function JsonTree({ data, depth = 0 }) {
  if (typeof data !== "object" || data === null) {
    return <span style={{ color: "#1971c2" }}>{String(data)}</span>;
  }

  return (
    <ul style={{ paddingLeft: depth === 0 ? 0 : 20, listStyle: "none" }}>
      {Object.entries(data).map(([key, value]) => (
        <li key={key} style={{ margin: "4px 0" }}>
          <strong style={{ color: "#e67700" }}>{key}:</strong>{" "}
          {typeof value === "object" && value !== null ? (
            <JsonTree data={value} depth={depth + 1} />
          ) : (
            <span style={{ color: "#1971c2" }}>{String(value)}</span>
          )}
        </li>
      ))}
    </ul>
  );
}

export default function App() {
  const data = {
    name: "Alice",
    age: 28,
    address: {
      city: "Bangalore",
      zip: "560001",
      country: "India",
    },
    skills: {
      frontend: "React",
      backend: "FastAPI",
    },
  };

  return (
    <div style={{ padding: 24 }}>
      <h2>Nested JSON Tree</h2>
      <JsonTree data={data} />
    </div>
  );
}
```

---

## 8. Fetch API Data and Render Card UI

```javascript
import { useState, useEffect } from "react";

function UserCard({ user }) {
  return (
    <div
      style={{
        border: "1px solid #ddd",
        borderRadius: 8,
        padding: 16,
        background: "#f9f9f9",
      }}
    >
      <h3 style={{ margin: "0 0 8px" }}>{user.name}</h3>
      <p style={{ margin: "4px 0", color: "#555" }}>📧 {user.email}</p>
      <p style={{ margin: "4px 0", color: "#555" }}>🌐 {user.website}</p>
      <p style={{ margin: "4px 0", color: "#555" }}>🏢 {user.company.name}</p>
    </div>
  );
}

export default function App() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    fetch("https://jsonplaceholder.typicode.com/users")
      .then((r) => r.json())
      .then((data) => { setUsers(data); setLoading(false); })
      .catch(() => { setError("Failed to load"); setLoading(false); });
  }, []);

  if (loading) return <div style={{ padding: 24 }}>Loading...</div>;
  if (error) return <div style={{ padding: 24, color: "red" }}>{error}</div>;

  return (
    <div style={{ padding: 24 }}>
      <h2>Users</h2>
      <div
        style={{
          display: "grid",
          gridTemplateColumns: "repeat(auto-fill, minmax(260px, 1fr))",
          gap: 16,
        }}
      >
        {users.map((user) => (
          <UserCard key={user.id} user={user} />
        ))}
      </div>
    </div>
  );
}
```

---

## 9. Edit / Delete / Add (Todo App)

```javascript
import { useState } from "react";

export default function App() {
  const [todos, setTodos] = useState([
    { id: 1, text: "Buy groceries" },
    { id: 2, text: "Read book" },
  ]);
  const [input, setInput] = useState("");
  const [editId, setEditId] = useState(null);
  const [editText, setEditText] = useState("");

  const add = () => {
    if (!input.trim()) return;
    setTodos([...todos, { id: Date.now(), text: input.trim() }]);
    setInput("");
  };

  const remove = (id) => setTodos(todos.filter((t) => t.id !== id));

  const startEdit = (todo) => { setEditId(todo.id); setEditText(todo.text); };

  const saveEdit = () => {
    setTodos(todos.map((t) => (t.id === editId ? { ...t, text: editText } : t)));
    setEditId(null);
  };

  return (
    <div style={{ padding: 24, maxWidth: 500 }}>
      <h2>Todo App</h2>

      <div style={{ display: "flex", gap: 8, marginBottom: 16 }}>
        <input
          value={input}
          onChange={(e) => setInput(e.target.value)}
          onKeyDown={(e) => e.key === "Enter" && add()}
          placeholder="Add a todo..."
          style={{ flex: 1, padding: 8 }}
        />
        <button onClick={add} style={{ padding: "8px 16px" }}>Add</button>
      </div>

      <ul style={{ listStyle: "none", padding: 0 }}>
        {todos.map((todo) => (
          <li
            key={todo.id}
            style={{
              display: "flex",
              alignItems: "center",
              gap: 8,
              padding: "8px 0",
              borderBottom: "1px solid #eee",
            }}
          >
            {editId === todo.id ? (
              <>
                <input
                  value={editText}
                  onChange={(e) => setEditText(e.target.value)}
                  style={{ flex: 1, padding: 6 }}
                />
                <button onClick={saveEdit}>Save</button>
                <button onClick={() => setEditId(null)}>Cancel</button>
              </>
            ) : (
              <>
                <span style={{ flex: 1 }}>{todo.text}</span>
                <button onClick={() => startEdit(todo)}>Edit</button>
                <button onClick={() => remove(todo.id)} style={{ color: "red" }}>Delete</button>
              </>
            )}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

---

## 10. Form with Validation

```javascript
import { useState } from "react";

export default function App() {
  const [form, setForm] = useState({ email: "", password: "" });
  const [errors, setErrors] = useState({});
  const [submitted, setSubmitted] = useState(false);
  const [submitting, setSubmitting] = useState(false);

  const validate = () => {
    const errs = {};
    if (!form.email) errs.email = "Email is required";
    else if (!/\S+@\S+\.\S+/.test(form.email)) errs.email = "Invalid email format";
    if (!form.password) errs.password = "Password is required";
    else if (form.password.length < 6) errs.password = "Minimum 6 characters";
    return errs;
  };

  const handleChange = (e) => {
    const { name, value } = e.target;
    setForm((f) => ({ ...f, [name]: value }));
    if (errors[name]) setErrors((e) => ({ ...e, [name]: "" }));
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    const errs = validate();
    if (Object.keys(errs).length) { setErrors(errs); return; }
    setSubmitting(true);
    await new Promise((r) => setTimeout(r, 1000)); // simulate API call
    setSubmitting(false);
    setSubmitted(true);
  };

  if (submitted) return (
    <div style={{ padding: 24, color: "green" }}>
      ✅ Submitted successfully!
      <button onClick={() => { setSubmitted(false); setForm({ email: "", password: "" }); }} style={{ marginLeft: 12 }}>
        Reset
      </button>
    </div>
  );

  return (
    <div style={{ padding: 24, maxWidth: 400 }}>
      <h2>Login Form</h2>
      <form onSubmit={handleSubmit}>
        <div style={{ marginBottom: 16 }}>
          <label style={{ display: "block", marginBottom: 4 }}>Email</label>
          <input
            name="email"
            value={form.email}
            onChange={handleChange}
            placeholder="email@example.com"
            style={{ width: "100%", padding: 8, boxSizing: "border-box" }}
          />
          {errors.email && <span style={{ color: "red", fontSize: 13 }}>{errors.email}</span>}
        </div>

        <div style={{ marginBottom: 16 }}>
          <label style={{ display: "block", marginBottom: 4 }}>Password</label>
          <input
            name="password"
            type="password"
            value={form.password}
            onChange={handleChange}
            placeholder="Min 6 characters"
            style={{ width: "100%", padding: 8, boxSizing: "border-box" }}
          />
          {errors.password && <span style={{ color: "red", fontSize: 13 }}>{errors.password}</span>}
        </div>

        <button
          type="submit"
          disabled={submitting}
          style={{ padding: "10px 24px", background: "#1971c2", color: "#fff", border: "none", borderRadius: 4, cursor: "pointer" }}
        >
          {submitting ? "Submitting..." : "Login"}
        </button>
      </form>
    </div>
  );
}
```

---

## 11. Dynamic React Components

```javascript
import { useState } from "react";

// Sub-components
function TextInput({ label, value, onChange }) {
  return (
    <div style={{ marginBottom: 12 }}>
      <label style={{ display: "block", marginBottom: 4 }}>{label}</label>
      <input value={value} onChange={(e) => onChange(e.target.value)} style={{ padding: 8, width: "100%" }} />
    </div>
  );
}

function SelectInput({ label, value, onChange }) {
  return (
    <div style={{ marginBottom: 12 }}>
      <label style={{ display: "block", marginBottom: 4 }}>{label}</label>
      <select value={value} onChange={(e) => onChange(e.target.value)} style={{ padding: 8, width: "100%" }}>
        <option value="">Select...</option>
        <option value="admin">Admin</option>
        <option value="user">User</option>
        <option value="guest">Guest</option>
      </select>
    </div>
  );
}

function CheckboxInput({ label, value, onChange }) {
  return (
    <div style={{ marginBottom: 12, display: "flex", alignItems: "center", gap: 8 }}>
      <input type="checkbox" checked={!!value} onChange={(e) => onChange(e.target.checked)} />
      <label>{label}</label>
    </div>
  );
}

// Component map
const COMPONENTS = {
  text: TextInput,
  select: SelectInput,
  checkbox: CheckboxInput,
};

function DynamicForm({ fields }) {
  const [values, setValues] = useState({});

  const handleChange = (name, value) => {
    setValues((v) => ({ ...v, [name]: value }));
  };

  return (
    <form onSubmit={(e) => { e.preventDefault(); alert(JSON.stringify(values, null, 2)); }}>
      {fields.map((field) => {
        const Component = COMPONENTS[field.type];
        if (!Component) return null;
        return (
          <Component
            key={field.name}
            label={field.label}
            value={values[field.name] || ""}
            onChange={(val) => handleChange(field.name, val)}
          />
        );
      })}
      <button type="submit" style={{ padding: "8px 20px" }}>Submit</button>
    </form>
  );
}

export default function App() {
  const fields = [
    { name: "username", type: "text", label: "Username" },
    { name: "role", type: "select", label: "Role" },
    { name: "active", type: "checkbox", label: "Active" },
  ];

  return (
    <div style={{ padding: 24, maxWidth: 400 }}>
      <h2>Dynamic Form</h2>
      <DynamicForm fields={fields} />
    </div>
  );
}
```
