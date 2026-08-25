# JavaScript Fundamentals — Round 3 Prep (Uber)

---

## 1. Hoisting

JavaScript moves **declarations** to the top of their scope before execution.

```javascript
console.log(x); // undefined (not error)
var x = 5;

// What JS actually does:
var x;           // declaration hoisted
console.log(x);  // undefined
x = 5;           // assignment stays
```

**`let` and `const` are hoisted but NOT initialized — Temporal Dead Zone (TDZ):**
```javascript
console.log(y); // ReferenceError: Cannot access 'y' before initialization
let y = 5;
```

**Functions are fully hoisted:**
```javascript
greet(); // "Hello" — works fine
function greet() { console.log("Hello"); }

// But function expressions are NOT:
sayHi(); // TypeError: sayHi is not a function
var sayHi = function() { console.log("Hi"); };
```

---

## 2. var, let, const & Scopes

**var — function scoped:**
```javascript
function example() {
    if (true) {
        var x = 10; // function scoped, not block scoped
    }
    console.log(x); // 10 — accessible outside the if block
}
```

**let — block scoped:**
```javascript
function example() {
    if (true) {
        let x = 10; // block scoped
    }
    console.log(x); // ReferenceError — not accessible outside block
}
```

**const — block scoped, cannot be reassigned:**
```javascript
const x = 10;
x = 20; // TypeError

// But objects/arrays CAN be mutated:
const obj = { name: "Alice" };
obj.name = "Bob"; // allowed — you're mutating, not reassigning
obj = {};         // TypeError — can't reassign
```

**Scope types:**
- **Global scope** — accessible everywhere
- **Function scope** — accessible inside function only
- **Block scope** — accessible inside {} only (let/const)

---

## 3. Closures & Lexical Scope

**Lexical scope** — a function can access variables from its outer scope at the time it was **defined**, not called.

**Closure** — a function that remembers variables from its outer scope even after the outer function has returned.

```javascript
function counter() {
    let count = 0;           // outer variable

    return function() {      // inner function — closure
        count++;
        return count;
    };
}

const increment = counter();
console.log(increment()); // 1
console.log(increment()); // 2
console.log(increment()); // 3
// count is preserved in memory because of closure
```

**Real-world use — private variables:**
```javascript
function createUser(name) {
    let _password = "secret"; // private

    return {
        getName: () => name,
        checkPassword: (input) => input === _password
    };
}

const user = createUser("Alice");
console.log(user.getName());          // "Alice"
console.log(user.checkPassword("secret")); // true
console.log(user._password);         // undefined — truly private
```

**Classic closure gotcha:**
```javascript
// Wrong — all log 3
for (var i = 0; i < 3; i++) {
    setTimeout(() => console.log(i), 1000);
}

// Fix 1 — use let (block scope)
for (let i = 0; i < 3; i++) {
    setTimeout(() => console.log(i), 1000); // 0, 1, 2
}

// Fix 2 — IIFE closure
for (var i = 0; i < 3; i++) {
    (function(j) {
        setTimeout(() => console.log(j), 1000);
    })(i);
}
```

---

## 4. Event Loop, Call Stack, Microtasks/Macrotasks

**Call Stack** — executes synchronous code, one function at a time (LIFO).

**Web APIs** — handle async operations (setTimeout, fetch, DOM events) outside the call stack.

**Task Queue (Macrotask Queue)** — holds callbacks from setTimeout, setInterval, DOM events.

**Microtask Queue** — holds Promise callbacks (.then, .catch), queueMicrotask. **Always runs before macrotasks.**

**Event Loop** — continuously checks: if call stack is empty → run all microtasks → run one macrotask → repeat.

```javascript
console.log("1 - sync");

setTimeout(() => console.log("2 - macrotask"), 0);

Promise.resolve().then(() => console.log("3 - microtask"));

console.log("4 - sync");

// Output:
// 1 - sync
// 4 - sync
// 3 - microtask   ← microtasks run before macrotasks
// 2 - macrotask
```

**Visual:**
```
Call Stack        Microtask Queue     Macrotask Queue
──────────        ───────────────     ───────────────
main()            Promise.then()      setTimeout cb
console.log(1)
console.log(4)
               ↓ stack empty
               run all microtasks
               console.log(3)
               ↓ microtasks empty
               run one macrotask
               console.log(2)
```

---

## 5. Promises & Promise Methods

A Promise represents a value that will be available in the future.

**States:** pending → fulfilled OR rejected

```javascript
const promise = new Promise((resolve, reject) => {
    setTimeout(() => resolve("done"), 1000);
});

promise
    .then(value => console.log(value))  // "done"
    .catch(err => console.log(err))
    .finally(() => console.log("always runs"));
```

**Promise methods:**

```javascript
// Promise.all — all must succeed, fails if any fails
Promise.all([fetch(url1), fetch(url2), fetch(url3)])
    .then(([r1, r2, r3]) => console.log(r1, r2, r3));

// Promise.allSettled — waits for all, never fails
Promise.allSettled([fetch(url1), fetch(url2)])
    .then(results => results.forEach(r => console.log(r.status)));
// { status: "fulfilled", value: ... }
// { status: "rejected", reason: ... }

// Promise.race — first one to settle wins
Promise.race([fetch(url1), fetch(url2)])
    .then(first => console.log("fastest:", first));

// Promise.any — first one to SUCCEED wins (ignores rejections)
Promise.any([failingPromise, fetch(url2)])
    .then(first => console.log(first));
```

**When to use which:**
| Method | Use Case |
|---|---|
| `Promise.all` | All requests must succeed (parallel fetch) |
| `Promise.allSettled` | Run all, handle each result individually |
| `Promise.race` | Timeout pattern — first to respond wins |
| `Promise.any` | Try multiple sources, use first success |

---

## 6. Promise Chaining & Promise Hell

**Promise chaining:**
```javascript
fetch("/user")
    .then(res => res.json())
    .then(user => fetch(`/orders/${user.id}`))
    .then(res => res.json())
    .then(orders => console.log(orders))
    .catch(err => console.log(err));
```

**Promise Hell (callback hell with promises):**
```javascript
// Nested promises — hard to read
fetch("/user").then(res => {
    res.json().then(user => {
        fetch(`/orders/${user.id}`).then(res => {
            res.json().then(orders => {
                console.log(orders); // deeply nested
            });
        });
    });
});
```

**Fix: async/await (next section) or flat chaining (always return promises).**

---

## 7. async/await & Error Handling

`async/await` is syntactic sugar over Promises. Makes async code look synchronous.

```javascript
async function fetchUser() {
    const res = await fetch("/user");     // waits for promise
    const user = await res.json();
    return user;
}
```

**Error handling:**
```javascript
async function fetchData() {
    try {
        const res = await fetch("/user");
        if (!res.ok) throw new Error("Request failed");
        const data = await res.json();
        return data;
    } catch (err) {
        console.error("Error:", err.message);
    } finally {
        console.log("always runs");
    }
}
```

**Multiple parallel requests:**
```javascript
async function fetchAll() {
    // Sequential — slow (waits for each)
    const user = await fetch("/user");
    const orders = await fetch("/orders");

    // Parallel — fast (both start at same time)
    const [user, orders] = await Promise.all([
        fetch("/user"),
        fetch("/orders")
    ]);
}
```

---

## 8. `this` Keyword

`this` refers to the object that is **calling** the function.

```javascript
// In object method — this = the object
const user = {
    name: "Alice",
    greet() {
        console.log(this.name); // "Alice"
    }
};

// In regular function — this = global (window) or undefined (strict mode)
function greet() {
    console.log(this); // window / undefined
}

// Arrow function — this = inherited from enclosing scope
const user = {
    name: "Alice",
    greet: () => {
        console.log(this.name); // undefined — arrow fn has no own 'this'
    }
};

// In class — this = instance
class User {
    constructor(name) { this.name = name; }
    greet() { console.log(this.name); } // instance's name
}
```

---

## 9. call, apply, bind

All three let you **explicitly set `this`**.

```javascript
function greet(greeting, punctuation) {
    console.log(`${greeting}, ${this.name}${punctuation}`);
}

const user = { name: "Alice" };

// call — invoke immediately, args passed individually
greet.call(user, "Hello", "!");       // "Hello, Alice!"

// apply — invoke immediately, args passed as array
greet.apply(user, ["Hello", "!"]);    // "Hello, Alice!"

// bind — returns a new function with 'this' fixed, invoke later
const boundGreet = greet.bind(user, "Hello");
boundGreet("!");  // "Hello, Alice!"
```

**Real-world use of bind:**
```javascript
class Button {
    constructor() {
        this.count = 0;
        this.handleClick = this.handleClick.bind(this); // fix 'this' for event handler
    }
    handleClick() {
        this.count++;
    }
}
```

---

## 10. Deep Copy vs Shallow Copy

**Shallow copy** — copies top-level properties. Nested objects are still referenced.

```javascript
const original = { name: "Alice", address: { city: "Bangalore" } };

// Shallow copy methods:
const copy1 = { ...original };
const copy2 = Object.assign({}, original);

copy1.name = "Bob";           // original.name unchanged
copy1.address.city = "Mumbai"; // original.address.city ALSO changes!
```

**Deep copy** — copies everything recursively. No shared references.

```javascript
// Method 1: JSON (simple but loses functions, undefined, Date)
const deep = JSON.parse(JSON.stringify(original));

// Method 2: structuredClone (modern, handles more types)
const deep = structuredClone(original);

// Method 3: Lodash
import _ from "lodash";
const deep = _.cloneDeep(original);
```

---

## 11. Objects, Map & Key Types

**Object keys are always strings (or Symbols):**
```javascript
const obj = {};
obj[1] = "one";
obj[true] = "bool";
console.log(Object.keys(obj)); // ["1", "true"] — converted to string
```

**Map — keys can be ANY type:**
```javascript
const map = new Map();
map.set(1, "number key");
map.set(true, "boolean key");
map.set({ id: 1 }, "object key");

console.log(map.get(1));    // "number key"
console.log(map.size);      // 3
```

**When to use Map over Object:**
- Keys are not strings
- Need to preserve insertion order
- Frequent add/delete (Map is faster)
- Need `.size` property

---

## 12. for...in vs for...of

**for...in — iterates over keys (enumerable properties):**
```javascript
const obj = { a: 1, b: 2, c: 3 };
for (const key in obj) {
    console.log(key); // "a", "b", "c"
}

// Also works on arrays (but gives string indices — avoid this)
const arr = [10, 20, 30];
for (const i in arr) {
    console.log(i); // "0", "1", "2" — string keys!
}
```

**for...of — iterates over values (iterables: arrays, strings, Map, Set):**
```javascript
const arr = [10, 20, 30];
for (const val of arr) {
    console.log(val); // 10, 20, 30
}

const str = "hello";
for (const char of str) {
    console.log(char); // h, e, l, l, o
}

// Does NOT work on plain objects (not iterable)
for (const val of { a: 1 }) {} // TypeError
```

---

## 13. Memoization & Debounce

### Memoization
Cache the result of expensive function calls.

```javascript
function memoize(fn) {
    const cache = {};
    return function(...args) {
        const key = JSON.stringify(args);
        if (cache[key] !== undefined) {
            console.log("from cache");
            return cache[key];
        }
        cache[key] = fn(...args);
        return cache[key];
    };
}

const expensiveAdd = memoize((a, b) => {
    console.log("computing...");
    return a + b;
});

expensiveAdd(2, 3); // computing... 5
expensiveAdd(2, 3); // from cache   5
```

### Debounce
Delays execution until after N ms of inactivity. Used for search inputs, resize handlers.

```javascript
function debounce(fn, delay) {
    let timer;
    return function(...args) {
        clearTimeout(timer);
        timer = setTimeout(() => {
            fn.apply(this, args);
        }, delay);
    };
}

// Usage
const handleSearch = debounce((query) => {
    fetch(`/search?q=${query}`);
}, 500);

input.addEventListener("input", (e) => handleSearch(e.target.value));
// API only called 500ms after user stops typing
```

### Throttle (bonus — often asked alongside debounce)
Ensures function runs at most once per N ms. Used for scroll, resize.

```javascript
function throttle(fn, limit) {
    let lastCall = 0;
    return function(...args) {
        const now = Date.now();
        if (now - lastCall >= limit) {
            lastCall = now;
            fn.apply(this, args);
        }
    };
}
```

**Debounce vs Throttle:**
| | Debounce | Throttle |
|---|---|---|
| Fires | After inactivity | At regular intervals |
| Use case | Search input | Scroll, resize |
