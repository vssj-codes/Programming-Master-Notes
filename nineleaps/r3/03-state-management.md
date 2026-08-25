# State Management — Round 3 Prep (Uber)

---

## 1. Redux Fundamentals & How It Works Internally

### Core Concept
Redux is a predictable state container. The entire app state lives in one **store**. State is read-only — you change it by dispatching **actions**, which are handled by **reducers**.

```
UI → dispatch(action) → reducer → new state → UI re-renders
```

### Core Pieces

**Action** — plain object describing what happened:
```javascript
{ type: "INCREMENT", payload: 1 }
```

**Reducer** — pure function, takes current state + action, returns new state:
```javascript
function counterReducer(state = { count: 0 }, action) {
    switch (action.type) {
        case "INCREMENT":
            return { count: state.count + action.payload };
        case "DECREMENT":
            return { count: state.count - action.payload };
        default:
            return state;
    }
}
```

**Store** — holds the state tree:
```javascript
import { createStore } from "redux";
const store = createStore(counterReducer);

store.getState();      // { count: 0 }
store.dispatch({ type: "INCREMENT", payload: 1 });
store.getState();      // { count: 1 }
store.subscribe(() => console.log(store.getState())); // listen to changes
```

---

### Redux Toolkit (Modern Redux — always use this)

Redux Toolkit eliminates boilerplate.

```javascript
import { createSlice, configureStore } from "@reduxjs/toolkit";

// Slice = actions + reducer in one
const counterSlice = createSlice({
    name: "counter",
    initialState: { count: 0 },
    reducers: {
        increment: (state, action) => {
            state.count += action.payload; // Immer allows mutation syntax
        },
        decrement: (state, action) => {
            state.count -= action.payload;
        }
    }
});

export const { increment, decrement } = counterSlice.actions;

const store = configureStore({
    reducer: { counter: counterSlice.reducer }
});
```

**In React component:**
```javascript
import { useSelector, useDispatch } from "react-redux";
import { increment, decrement } from "./counterSlice";

function Counter() {
    const count = useSelector(state => state.counter.count);
    const dispatch = useDispatch();

    return (
        <div>
            <p>{count}</p>
            <button onClick={() => dispatch(increment(1))}>+</button>
            <button onClick={() => dispatch(decrement(1))}>-</button>
        </div>
    );
}
```

**Provider setup:**
```javascript
import { Provider } from "react-redux";
import store from "./store";

function App() {
    return (
        <Provider store={store}>
            <Counter />
        </Provider>
    );
}
```

---

### How Redux Works Internally

1. `createStore(reducer)` creates the store with initial state
2. `store.dispatch(action)` calls `reducer(currentState, action)` → returns new state
3. Store notifies all subscribers (`store.subscribe`)
4. React-Redux's `useSelector` subscribes to store — re-renders component when selected state changes
5. Redux Toolkit uses **Immer** under the hood — lets you write "mutating" code that actually produces immutable updates

---

### Async Actions — Redux Thunk

Redux Toolkit includes `createAsyncThunk` for async operations:

```javascript
export const fetchUser = createAsyncThunk("user/fetch", async (userId) => {
    const res = await fetch(`/api/users/${userId}`);
    return res.json();
});

const userSlice = createSlice({
    name: "user",
    initialState: { data: null, status: "idle", error: null },
    extraReducers: (builder) => {
        builder
            .addCase(fetchUser.pending, (state) => {
                state.status = "loading";
            })
            .addCase(fetchUser.fulfilled, (state, action) => {
                state.status = "succeeded";
                state.data = action.payload;
            })
            .addCase(fetchUser.rejected, (state, action) => {
                state.status = "failed";
                state.error = action.error.message;
            });
    }
});
```

---

## 2. Redux vs Context API

| | Context API | Redux |
|---|---|---|
| Setup | Built-in, zero config | External library, boilerplate |
| Best for | Low-frequency updates (theme, auth, language) | Complex, high-frequency state |
| Performance | All consumers re-render on any change | `useSelector` only re-renders if selected value changed |
| DevTools | No | Yes — time travel, action log |
| Middleware | No | Yes (thunk, saga, logger) |
| Async | Manual | Built-in (createAsyncThunk) |
| Scale | Small-medium apps | Large apps |

**Decision:**
- Auth state, theme, language → Context API
- Complex shared state, frequent updates, many components → Redux

---

## 3. Redux Optimization & Reducing Bundle Size

### Selector Optimization with Reselect
```javascript
import { createSelector } from "@reduxjs/toolkit";

// Memoized selector — only recomputes when input changes
const selectCompletedTodos = createSelector(
    state => state.todos,
    todos => todos.filter(todo => todo.completed) // expensive filter
);

// In component:
const completedTodos = useSelector(selectCompletedTodos);
```

### Reducing Bundle Size
- **Code splitting** — only load Redux slices when needed
- **RTK Query** — replaces manual async thunks + caching layer built in
- **Normalize state** — use `createEntityAdapter` for flat, normalized state (avoids duplicate data)

```javascript
import { createEntityAdapter, createSlice } from "@reduxjs/toolkit";

const usersAdapter = createEntityAdapter();
// stores as { ids: [1,2,3], entities: { 1: {...}, 2: {...} } }
// makes lookups O(1) instead of O(n)

const usersSlice = createSlice({
    name: "users",
    initialState: usersAdapter.getInitialState(),
    reducers: {
        addUser: usersAdapter.addOne,
        removeUser: usersAdapter.removeOne,
        updateUser: usersAdapter.updateOne
    }
});
```

### Avoiding Unnecessary Re-renders
```javascript
// Bad — new object reference on every render
const data = useSelector(state => ({
    user: state.user,
    count: state.counter.count
}));

// Good — select primitives or use shallowEqual
import { shallowEqual } from "react-redux";
const { user, count } = useSelector(
    state => ({ user: state.user, count: state.counter.count }),
    shallowEqual
);
```

---

## 4. Practical Redux Usage

### Project Structure
```
src/
  store/
    index.js          ← configureStore
  features/
    counter/
      counterSlice.js
    user/
      userSlice.js
      userThunks.js
```

### RTK Query (Modern approach for API calls)
```javascript
import { createApi, fetchBaseQuery } from "@reduxjs/toolkit/query/react";

export const api = createApi({
    reducerPath: "api",
    baseQuery: fetchBaseQuery({ baseUrl: "/api" }),
    endpoints: (builder) => ({
        getUser: builder.query({
            query: (id) => `/users/${id}`,
        }),
        createUser: builder.mutation({
            query: (body) => ({ url: "/users", method: "POST", body }),
        }),
    }),
});

export const { useGetUserQuery, useCreateUserMutation } = api;

// In component:
function UserProfile({ id }) {
    const { data, isLoading, error } = useGetUserQuery(id);
    if (isLoading) return <Spinner />;
    if (error) return <Error />;
    return <div>{data.name}</div>;
}
```

RTK Query handles caching, refetching, loading/error states automatically.
