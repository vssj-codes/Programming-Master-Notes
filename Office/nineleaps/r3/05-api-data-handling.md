# API & Data Handling — Round 3 Prep (Uber)

---

## 1. Fetching Data from APIs

**Native fetch:**
```javascript
async function fetchUser(id) {
    const res = await fetch(`/api/users/${id}`);
    if (!res.ok) throw new Error(`HTTP error: ${res.status}`);
    return res.json();
}
```

**Axios (common in projects):**
```javascript
import axios from "axios";

const api = axios.create({
    baseURL: "https://api.example.com",
    timeout: 5000,
    headers: { "Content-Type": "application/json" }
});

// Request interceptor — add auth token
api.interceptors.request.use(config => {
    config.headers.Authorization = `Bearer ${getToken()}`;
    return config;
});

// Response interceptor — handle errors globally
api.interceptors.response.use(
    res => res.data,
    err => {
        if (err.response?.status === 401) logout();
        return Promise.reject(err);
    }
);
```

---

## 2. API Integration & Rendering Data

**Standard pattern with loading/error states:**
```javascript
function UserList() {
    const [users, setUsers] = useState([]);
    const [loading, setLoading] = useState(true);
    const [error, setError] = useState(null);

    useEffect(() => {
        fetch("/api/users")
            .then(res => res.json())
            .then(data => setUsers(data))
            .catch(err => setError(err.message))
            .finally(() => setLoading(false));
    }, []);

    if (loading) return <Spinner />;
    if (error) return <Error message={error} />;
    return <ul>{users.map(u => <li key={u.id}>{u.name}</li>)}</ul>;
}
```

---

## 3. Multiple API Handling & Data Flow

**Parallel requests:**
```javascript
async function fetchDashboardData() {
    const [users, orders, analytics] = await Promise.all([
        fetch("/api/users").then(r => r.json()),
        fetch("/api/orders").then(r => r.json()),
        fetch("/api/analytics").then(r => r.json()),
    ]);
    return { users, orders, analytics };
}
```

**Sequential requests (when one depends on another):**
```javascript
async function fetchUserOrders(userId) {
    const user = await fetch(`/api/users/${userId}`).then(r => r.json());
    const orders = await fetch(`/api/orders?userId=${user.id}`).then(r => r.json());
    return { user, orders };
}
```

**Independent requests with separate error handling:**
```javascript
const results = await Promise.allSettled([
    fetch("/api/users"),
    fetch("/api/orders"),
]);

results.forEach((result, i) => {
    if (result.status === "fulfilled") handleSuccess(result.value);
    else handleError(result.reason);
});
```

---

## 4. gRPC Basics & Fetching Data

**What is gRPC?**
- Google Remote Procedure Call — protocol for service-to-service communication
- Uses **Protocol Buffers** (binary format) instead of JSON — faster, smaller
- Strongly typed — defined via `.proto` files
- Supports streaming (server → client, client → server, bidirectional)

**Protocol Buffers example:**
```proto
// user.proto
syntax = "proto3";

message User {
    int32 id = 1;
    string name = 2;
    string email = 3;
}

service UserService {
    rpc GetUser (GetUserRequest) returns (User);
    rpc ListUsers (Empty) returns (stream User); // server streaming
}
```

**In browser (gRPC-Web):**
Direct gRPC from browser isn't supported — needs **gRPC-Web** + a proxy (Envoy).

```javascript
import { UserServiceClient } from "./generated/UserServiceClientPb";
import { GetUserRequest } from "./generated/user_pb";

const client = new UserServiceClient("https://api.example.com");
const request = new GetUserRequest();
request.setId(123);

client.getUser(request, {}, (err, response) => {
    if (err) console.error(err);
    else console.log(response.getName());
});
```

**gRPC vs REST:**
| | REST | gRPC |
|---|---|---|
| Format | JSON (text) | Protocol Buffers (binary) |
| Speed | Slower | Faster (~5-7x) |
| Browser support | Native | Needs gRPC-Web + proxy |
| Streaming | Limited | Built-in |
| Use case | Public APIs | Internal microservices |

---

## 5. GraphQL Basics

**What is GraphQL?**
A query language for APIs. Client specifies exactly what data it needs — no over-fetching or under-fetching.

**REST problem:**
```
GET /user/1        → returns ALL user fields (over-fetching)
GET /user/1/orders → separate request needed (under-fetching)
```

**GraphQL solution:**
```graphql
query {
    user(id: 1) {
        name          # only what we need
        email
        orders {
            id
            total
        }
    }
}
# One request, exactly the data needed
```

**Queries, Mutations, Subscriptions:**
```graphql
# Query — read data
query GetUser($id: ID!) {
    user(id: $id) { name email }
}

# Mutation — write data
mutation CreateUser($name: String!, $email: String!) {
    createUser(name: $name, email: $email) { id name }
}

# Subscription — real-time updates
subscription OnOrderUpdate($orderId: ID!) {
    orderUpdated(id: $orderId) { status }
}
```

**In React with Apollo Client:**
```javascript
import { useQuery, useMutation } from "@apollo/client";
import { GET_USER, CREATE_USER } from "./queries";

function UserProfile({ id }) {
    const { data, loading, error } = useQuery(GET_USER, {
        variables: { id }
    });

    const [createUser] = useMutation(CREATE_USER);

    if (loading) return <Spinner />;
    return <div>{data.user.name}</div>;
}
```

---

## 6. TanStack Query

Powerful async state management for server data. Handles caching, background refetching, pagination, optimistic updates.

**Basic usage:**
```javascript
import { useQuery, useMutation, QueryClient, QueryClientProvider } from "@tanstack/react-query";

// Setup
const queryClient = new QueryClient();

function App() {
    return (
        <QueryClientProvider client={queryClient}>
            <UserList />
        </QueryClientProvider>
    );
}

// Query
function UserList() {
    const { data, isLoading, error, isFetching } = useQuery({
        queryKey: ["users"],
        queryFn: () => fetch("/api/users").then(r => r.json()),
        staleTime: 5 * 60 * 1000,  // data fresh for 5 min
        cacheTime: 10 * 60 * 1000, // cache kept for 10 min
    });

    if (isLoading) return <Spinner />;
    if (error) return <Error />;
    return (
        <ul>
            {isFetching && <span>Refreshing...</span>}
            {data.map(u => <li key={u.id}>{u.name}</li>)}
        </ul>
    );
}

// Mutation
function CreateUser() {
    const queryClient = useQueryClient();

    const mutation = useMutation({
        mutationFn: (newUser) => fetch("/api/users", {
            method: "POST",
            body: JSON.stringify(newUser)
        }).then(r => r.json()),
        onSuccess: () => {
            queryClient.invalidateQueries({ queryKey: ["users"] }); // refetch list
        }
    });

    return (
        <button onClick={() => mutation.mutate({ name: "Alice" })}>
            {mutation.isPending ? "Creating..." : "Create User"}
        </button>
    );
}
```

**Key features:**
- **Automatic caching** — same queryKey = cached result
- **Background refetching** — refreshes stale data automatically
- **Deduplication** — multiple components with same query = one request
- **Optimistic updates** — update UI before server confirms

---

## 7. API Loading/Error States

**Standard pattern:**
```javascript
function Component() {
    const { data, isLoading, isError, error, isFetching } = useQuery(...);

    if (isLoading) return <FullPageSpinner />;
    if (isError) return <ErrorBoundary message={error.message} />;

    return (
        <div>
            {isFetching && <TopLoadingBar />} {/* background refresh indicator */}
            <DataDisplay data={data} />
        </div>
    );
}
```

**Error boundary for unexpected errors:**
```javascript
class ErrorBoundary extends React.Component {
    state = { hasError: false };

    static getDerivedStateFromError() {
        return { hasError: true };
    }

    render() {
        if (this.state.hasError) return <h1>Something went wrong.</h1>;
        return this.props.children;
    }
}

<ErrorBoundary>
    <UserProfile />
</ErrorBoundary>
```

---

## 8. Authorization Handling

**JWT-based auth flow:**
```javascript
// Store token after login
localStorage.setItem("token", response.token);

// Attach to all requests via Axios interceptor
api.interceptors.request.use(config => {
    const token = localStorage.getItem("token");
    if (token) config.headers.Authorization = `Bearer ${token}`;
    return config;
});

// Handle 401 — redirect to login
api.interceptors.response.use(
    res => res,
    err => {
        if (err.response?.status === 401) {
            localStorage.removeItem("token");
            window.location.href = "/login";
        }
        return Promise.reject(err);
    }
);
```

**Protected routes in React:**
```javascript
function ProtectedRoute({ children }) {
    const token = localStorage.getItem("token");
    if (!token) return <Navigate to="/login" replace />;
    return children;
}

// Usage
<Route path="/dashboard" element={
    <ProtectedRoute>
        <Dashboard />
    </ProtectedRoute>
} />
```

**Auth context pattern:**
```javascript
const AuthContext = createContext(null);

function AuthProvider({ children }) {
    const [user, setUser] = useState(null);

    const login = async (credentials) => {
        const res = await api.post("/auth/login", credentials);
        setUser(res.data.user);
        localStorage.setItem("token", res.data.token);
    };

    const logout = () => {
        setUser(null);
        localStorage.removeItem("token");
    };

    return (
        <AuthContext.Provider value={{ user, login, logout }}>
            {children}
        </AuthContext.Provider>
    );
}

const useAuth = () => useContext(AuthContext);
```
