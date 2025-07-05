# Table-of-Contents

<!-- toc -->

- [1-React-Hook-useState](#1-react-hook-usestate)
  * [Syntax](#syntax)
  * [Key Points:](#key-points)
  * [Common mistakes:](#common-mistakes)
  * [Example:](#example)
- [2-React-Updating-Objects-in-State](#2-react-updating-objects-in-state)
  * [Key Points:](#key-points-1)
  * [Example:](#example-1)
- [3-React-Updating-Arrays-in-State](#3-react-updating-arrays-in-state)
  * [Key Points:](#key-points-2)
  * [Example:](#example-2)
- [4-React-Hook-useEffect](#4-react-hook-useeffect)
  * [Purpose:](#purpose)
  * [Parameters:](#parameters)
  * [Execution & Timing:](#execution--timing)
  * [Dependencies:](#dependencies)
  * [Cleanup function:](#cleanup-function)
  * [Example:](#example-3)
- [5-React-Hook-useContext](#5-react-hook-usecontext)
  * [Purpose:](#purpose-1)
  * [Parameters:](#parameters-1)
  * [Behavior & Usage:](#behavior--usage)
  * [Optimization Tips:](#optimization-tips)
- [6-React-Hook-useRef](#6-react-hook-useref)
  * [Purpose:](#purpose-2)
  * [Parameters:](#parameters-2)
  * [Behavior & Usage:](#behavior--usage-1)
  * [Example:](#example-4)
- [7-React-Hook-useReducer](#7-react-hook-usereducer)
  * [Purpose:](#purpose-3)
  * [Parameters:](#parameters-3)
  * [Returns:](#returns)
  * [Usage Patterns:](#usage-patterns)
- [8-Custom-Hooks](#8-custom-hooks)
  * [What-are-Custom-Hooks?](#what-are-custom-hooks)
  * [Writing Custom Hooks:](#writing-custom-hooks)
  * [Reusing Logic Between Components:](#reusing-logic-between-components)
- [9-React-Styling-CSS-Modules](#9-react-styling-css-modules)
  * [Scoped-CSS:](#scoped-css)
- [10-React-Styling-Styled-Components-Emotion](#10-react-styling-styled-components-emotion)
  * [Purpose:](#purpose-4)
  * [Example:](#example-5)
- [11-Tailwind-CSS-Integration](#11-tailwind-css-integration)
  * [Utility-first-CSS:](#utility-first-css)
  * [Example:](#example-6)
- [12-Performance-Optimization](#12-performance-optimization)
  * [React.memo (Component Memoization)](#reactmemo-component-memoization)
  * [useMemo & useCallback (Memoizing Values & Functions)](#usememo--usecallback-memoizing-values--functions)
  * [Code Splitting & Lazy Loading](#code-splitting--lazy-loading)
  * [Avoiding Unnecessary Re-renders](#avoiding-unnecessary-re-renders)
  * [Fetching Data with fetch / axios](#fetching-data-with-fetch--axios)

<!-- tocstop -->

---
## 1-React-Hook-useState

### Syntax
```js
const [state, setState] = useState(initialState);
````

### Key Points:

- Adds state to functional components (local state).
    
- `initialState` can be a value or lazy initializer function.
    
- Returns a pair: current state and setter function `[state, setState]`.
    
- Use `setState(newValue)` or functional update:
    
    ```js
    setState(prevState => newValue);
    ```
    
- State updates are asynchronous and batched for performance.
    
- Always treat state as immutable — replace objects/arrays instead of mutating.
    
- Using functional updater avoids stale closures in async updates.
    
- No re-render if state is set to the same value (shallow equality).
    
- `setState` function identity is stable across renders.
    

### Common mistakes:

- Calling `setState` inside render causes infinite loops.
    
- Initializer/updater functions run twice in Strict Mode (dev only).
    
- Follow hook rules: call only at component top-level, never inside loops or conditions.
    
- To reset state, use a unique key prop on the component.
    
- State only persists while the component is mounted.
    

### Example:

```js
function Counter() {
  const [count, setCount] = useState(0);
  return (
    <>
      <p>{count}</p>
      <button onClick={() => setCount(prev => prev + 1)}>Increment</button>
    </>
  );
}
```

---

## 2-React-Updating-Objects-in-State

### Key Points:

- React state is immutable — never mutate state objects directly.
    
- To update an object in state, create a new object by copying existing properties and overriding the ones to change.
    
- Use spread syntax or `Object.assign()` for shallow copying:
    
    ```js
    setState(prevState => ({
      ...prevState,
      updatedKey: newValue
    }));
    ```
    
- Using the functional updater form of `setState` ensures you have the latest state value and avoid stale closures.
    
- Nested objects require careful handling — shallow copy only copies top-level properties, deeper nested objects need manual copying or use of libraries like Immer for immutability.
    
- Avoid direct mutation like `state.obj.key = value` — this breaks React's state tracking and can cause no re-render or bugs.
    

### Example:

```js
const [user, setUser] = useState({ name: 'Alice', age: 25 });

// Update age without mutating user object
setUser(prevUser => ({
  ...prevUser,
  age: 26
}));
```

---

## 3-React-Updating-Arrays-in-State

### Key Points:

- React state is immutable — never mutate arrays directly.
    
- To update an array in state, create a new array instead of modifying the existing one.
    
- Use methods that return new arrays like spread operator, `concat`, `slice`, or array methods like `map`, `filter`, `reduce`.
    
- Use functional updater form of `setState` for accessing the latest state and avoiding stale closures:
    
    ```js
    setArray(prevArray => [...prevArray, newItem]); // Add item  
    setArray(prevArray => prevArray.filter(item => item.id !== idToRemove)); // Remove item  
    ```
    
- Avoid mutating methods like `push`, `pop`, `splice` directly on state arrays.
    
- React uses referential equality (`===`) to check if the array changed — must return a new array object for re-render.
    

### Example:

```js
const [todos, setTodos] = useState(['task1', 'task2']);

// Add a new todo
setTodos(prevTodos => [...prevTodos, 'task3']);

// Remove a todo
setTodos(prevTodos => prevTodos.filter(todo => todo !== 'task1'));
```

---

## 4-React-Hook-useEffect

### Purpose:

- Runs side effects (e.g., data fetching, subscriptions, DOM mutations) in functional components.
    

### Parameters:

- **Effect function (required):** runs after render; can return a cleanup function.
    
- **Dependencies array (optional):** controls when effect re-runs.
    

### Execution & Timing:

- Runs after browser paints UI (commit phase).
    
- Runs asynchronously, never during render phase.
    
- React batches multiple effects for performance.
    

### Dependencies:

- React compares dependencies using `Object.is` (shallow equality).
    
- Effect re-runs only when any dependency changes reference.
    
- If omitted, effect runs after every render.
    
- Empty array `[]` → runs once after mount (like `componentDidMount`).
    

### Cleanup function:

- Runs before the next effect execution and on component unmount.
    
- Used for unsubscribing, clearing timers, or cleanup side effects.
    

### Example:

```js
useEffect(() => {
  const timer = setInterval(() => {
    console.log("tick");
  }, 1000);

  return () => clearInterval(timer); // cleanup on unmount
}, []); // run once on mount
```

---

## 5-React-Hook-useContext

### Purpose:

- Access the current value of a React Context from within a functional component.
    

### Parameters:

- **Context object** created by `React.createContext()` (e.g., `SomeContext`).
    
- Returns the current context value for the nearest matching `<SomeContext.Provider>` above in the tree.
    

### Behavior & Usage:

- Reads context value from the nearest Provider above in the React tree.
    
- Context value can be any JavaScript value: primitives, objects, functions, etc.
    
- Supports dynamic updates: when Provider’s value changes, all consuming components re-render with the new value.
    

### Optimization Tips:

- Avoid passing new objects/functions inline as context value to prevent unnecessary re-renders.
    
- Memoize context values with `useMemo` or stable references to optimize performance.
    

---

## 6-React-Hook-useRef

### Purpose:

- Create a mutable ref object whose `.current` property persists across renders without causing re-renders.
    
- Commonly used to reference DOM elements or store mutable values that do not trigger UI updates.
    

### Parameters:

- `initialValue`: initial value assigned to `.current` property of the returned ref object.
    

### Behavior & Usage:

- Returns a ref object: `{ current: initialValue }`.
    
- The `.current` property can be read or mutated freely.
    
- Changing `.current` does NOT cause re-render.
    

### Example:

```js
<input ref={refContainer} />
// Access DOM node via refContainer.current after mount.
```

---

## 7-React-Hook-useReducer

### Purpose:

- Manage complex state logic in functional components using a reducer pattern.
    
- Alternative to `useState` for related state transitions or when next state depends on previous state.
    

### Parameters:

- **reducer (function):** `(state, action) => newState` — pure function describing how state changes based on action.
    
- **initialArg:** initial state value or argument for initializer.
    
- **init (optional):** function to lazily compute initial state from `initialArg`.
    

### Returns:

- `[state, dispatch]`:
    
    - `state`: current state.
        
    - `dispatch`: function to send action objects to reducer for state update.
        

### Usage Patterns:

- Use lazy initialization with `init` to optimize performance, especially if initial state is expensive to compute.
    

---

## 8-Custom-Hooks

### What-are-Custom-Hooks?

- Functions that start with `use` and call other Hooks internally.
    
- Enable reuse of stateful logic between components.
    
- Follow Hooks rules: only call Hooks at top level, and only from React function components or other Hooks.
    

### Writing Custom Hooks:

- Encapsulate reusable logic involving state, effects, context, refs, etc.
    
- Return any value or functions needed by consuming components.
    

### Reusing Logic Between Components:

- Custom Hooks extract common patterns to avoid duplication.
    

---

## 9-React-Styling-CSS-Modules

### Scoped-CSS:

- CSS class names scoped locally by default.
    
- Imported as JS object mapping class names to unique identifiers.
    
- Usage:
    
    ```js
    import styles from './Component.module.css';
    <div className={styles.container}></div>
    ```
    

---

## 10-React-Styling-Styled-Components-Emotion

### Purpose:

- CSS-in-JS libraries for styling React components.
    
- Define styles within JS using tagged template literals or object syntax.
    
- Supports dynamic styling via props, theming, and automatic vendor prefixing.
    

### Example:

```js
const Button = styled.button`
  background: ${props => props.primary ? 'blue' : 'gray'};
`;
```

---

## 11-Tailwind-CSS-Integration

### Utility-first-CSS:

- Integrate via PostCSS or CDN with React projects.
    
- Encourages rapid UI building with atomic classes.
    
- Can be combined with CSS Modules or CSS-in-JS.
    

### Example:

```js
<button className="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded">Click me</button>
```

---

## 12-Performance-Optimization

### React.memo (Component Memoization)

Purpose: Wraps a functional component to memoize its rendered output.

---

### useMemo & useCallback (Memoizing Values & Functions)

useMemo: Memoizes computed values to avoid expensive recalculations on each render.

useCallback: Memoizes function references to prevent child re-renders or effect retriggers.

---

### Code Splitting & Lazy Loading

Use React.lazy and `<Suspense>` to load components asynchronously.

---

### Avoiding Unnecessary Re-renders

React.memo, useMemo, and useCallback optimize rendering and computation.

---

### Fetching Data with fetch / axios

Fetch and axios for data fetching in React apps. React Query provides built-in caching and error handling.

---