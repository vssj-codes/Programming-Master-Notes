# Table-of-Contents

<!-- toc -->

- [1. Execution Context](#1-execution-context)
    + [What is an Execution Context?](#what-is-an-execution-context)
    + [Types of Execution Context:](#types-of-execution-context)
    + [Execution Context Phases:](#execution-context-phases)
    + [Call Stack (Execution Stack):](#call-stack-execution-stack)
    + [Key Concepts:](#key-concepts)
  * [2. 🎯 10 Interview Questions with Detailed Answers](#2-%F0%9F%8E%AF-10-interview-questions-with-detailed-answers)
    + [1. **Theory:**](#1-theory)
    + [2. **Code-Based Scenario:**](#2-code-based-scenario)
    + [3. **Code-Based Scenario (TDZ):**](#3-code-based-scenario-tdz)
    + [4. **Edge Case:**](#4-edge-case)
    + [5. **Real-World Bug:**](#5-real-world-bug)
    + [6. **Best Practices:**](#6-best-practices)
    + [7. **Debugging:**](#7-debugging)
    + [8. **Code Understanding:**](#8-code-understanding)
    + [9. **Closures + Execution Context:**](#9-closures--execution-context)
    + [10. **Memory Leak Case:**](#10-memory-leak-case)
  * [3. 🧠 Micro Notes (Quick Revision)](#3-%F0%9F%A7%A0-micro-notes-quick-revision)
  * [4. 💡 Demonstration Code Snippet](#4-%F0%9F%92%A1-demonstration-code-snippet)
- [2. Lexical Environment](#2-lexical-environment)
    + [What is a Lexical Environment?](#what-is-a-lexical-environment)
    + [Lexical vs Dynamic Scope:](#lexical-vs-dynamic-scope)
    + [Lexical Environment Chain:](#lexical-environment-chain)
    + [Key Concepts:](#key-concepts-1)
  * [2. 🎯 10 Interview Questions with Detailed Answers](#2-%F0%9F%8E%AF-10-interview-questions-with-detailed-answers-1)
    + [1. **Theory:**](#1-theory-1)
    + [2. **Code Understanding:**](#2-code-understanding)
    + [3. **Code Edge Case:**](#3-code-edge-case)
    + [4. **Closures and Lexical Scope:**](#4-closures-and-lexical-scope)
    + [5. **Nested Functions:**](#5-nested-functions)
    + [6. **Real-World Bug:**](#6-real-world-bug)
    + [7. **Debugging:**](#7-debugging-1)
    + [8. **Scope Chain Debugging:**](#8-scope-chain-debugging)
    + [9. **Arrow Functions + Lexical `this`:**](#9-arrow-functions--lexical-this)
    + [10. **Best Practices:**](#10-best-practices)
  * [3. 🧠 Micro Notes (Quick Revision)](#3-%F0%9F%A7%A0-micro-notes-quick-revision-1)
  * [4. 💡 Demonstration Code Snippet](#4-%F0%9F%92%A1-demonstration-code-snippet-1)

<!-- tocstop -->

---
# 1. Execution Context

### What is an Execution Context?

An **execution context** is the environment in which JavaScript code is evaluated and executed. It’s created whenever code is run.

### Types of Execution Context:

1. **Global Execution Context (GEC)**
    
    - Created by default when the JS file starts.
        
    - Creates `window` (in browsers) or `global` (in Node) object and `this`.
        
    - Variables declared with `var` are attached to `window`.
        
2. **Function Execution Context (FEC)**
    
    - Created every time a function is invoked.
        
    - Has its own `arguments`, `this`, local variables, scope chain.
        
3. **Eval Execution Context** (rarely used, avoid)
    
    - Created inside the `eval()` function.
        

### Execution Context Phases:

1. **Creation Phase**:
    
    - Scope chain is set.
        
    - `this` is determined.
        
    - Memory allocation for variables (`undefined` for `var`, TDZ for `let`/`const`).
        
    - Functions are hoisted.
        
2. **Execution Phase**:
    
    - Code is executed line-by-line.
        
    - Variables are assigned actual values.
        

### Call Stack (Execution Stack):

- A stack that manages function calls via LIFO (Last-In-First-Out).
    
- When a function is invoked, its context is pushed onto the stack.
    
- Once finished, it is popped off.
    

### Key Concepts:

- **Hoisting** only affects declarations (not initializations).
    
- **TDZ (Temporal Dead Zone)** is where `let`/`const` exist but are inaccessible before declaration.
    
- **`this`** depends on how a function is called, not where it’s defined.
    
- **Closures** are functions that access variables from their parent context.
    

---

## 2. 🎯 10 Interview Questions with Detailed Answers

---

### 1. **Theory:**

**Q:** What happens during the creation phase of an execution context?

**A:**

- Memory is allocated for variables and functions.
    
- Variables declared with `var` are hoisted and initialized as `undefined`.
    
- `let`/`const` are hoisted but left uninitialized (TDZ).
    
- Functions are hoisted with full definition.
    
- `this` is determined.
    
- Scope chain is initialized.
    

---

### 2. **Code-Based Scenario:**

**Q:** What is the output?

```js
console.log(a);
var a = 5;
```

**A:**  
Output: `undefined`  
`var a` is hoisted during the creation phase and assigned `undefined`. The assignment happens in the execution phase.

---

### 3. **Code-Based Scenario (TDZ):**

**Q:** What is the output?

```js
console.log(b);
let b = 10;
```

**A:**  
ReferenceError: Cannot access 'b' before initialization  
`let` is hoisted but not initialized — it resides in the Temporal Dead Zone until the declaration is evaluated.

---

### 4. **Edge Case:**

**Q:** Can you modify the value of `this` inside a regular function execution context?

**A:**  
No. In non-strict mode, `this` in global or regular functions refers to `window/global`. In strict mode, it’s `undefined`. Arrow functions don’t have their own `this`.

---

### 5. **Real-World Bug:**

**Q:** Why does this code throw an error?

```js
function example() {
  console.log(x);
  if (false) {
    let x = 20;
  }
}
example();
```

**A:**  
It throws a ReferenceError due to accessing `x` inside its TDZ — even though the condition is `false`, the block is still scoped and hoisted.

---

### 6. **Best Practices:**

**Q:** What are best practices for managing execution contexts in large applications?

**A:**

- Use strict mode (`'use strict'`) to catch errors.
    
- Avoid polluting the global scope.
    
- Use block scoping (`let`/`const`) to prevent variable leakage.
    
- Keep functions small to avoid deep call stacks.
    

---

### 7. **Debugging:**

**Q:** How can you debug execution context issues?

**A:**

- Use browser DevTools (Sources tab + Call Stack).
    
- Set breakpoints to observe the Call Stack and Scope.
    
- Watch for TDZ errors and improper `this` bindings.
    

---

### 8. **Code Understanding:**

**Q:** Explain the call stack for this code:

```js
function a() {
  b();
}
function b() {
  console.log('Hello');
}
a();
```

**A:**  
Call stack flow:

1. GEC created and pushed.
    
2. `a()` is called → FEC for `a` pushed.
    
3. `b()` is called inside `a` → FEC for `b` pushed.
    
4. `console.log` executes.
    
5. `b` and then `a` contexts are popped off.
    

---

### 9. **Closures + Execution Context:**

**Q:** Why does the following code work?

```js
function outer() {
  let count = 0;
  return function inner() {
    count++;
    return count;
  };
}
const counter = outer();
console.log(counter());
console.log(counter());
```

**A:**  
Because `inner()` retains access to the `outer()`'s execution context via closure, even after `outer()` has finished executing. It remembers `count`.

---

### 10. **Memory Leak Case:**

**Q:** How can poorly managed execution contexts lead to memory leaks?

**A:**  
If closures hold references to large objects from outer contexts unnecessarily, those objects aren’t garbage collected even if no longer needed.

---

## 3. 🧠 Micro Notes (Quick Revision)

- **Execution Context** = environment where code runs.
    
- **Types**: Global, Function, Eval.
    
- **Phases**:
    
    1. Creation → Hoisting, `this`, scope setup
        
    2. Execution → Code runs line-by-line
        
- **Call Stack** uses LIFO to manage function calls.
    
- **`var` hoisted** → `undefined`; `let/const` hoisted → TDZ.
    
- **`this`** is dynamic, based on function call, not definition.
    
- **Closures** retain reference to parent context.
    
- **Strict mode** helps avoid accidental global leakage.
    
- **Memory leaks** can happen with unintentional closures.
    

---

## 4. 💡 Demonstration Code Snippet
- Simple Eg
```javascript
// GEC, FEC

console.log(this)
console.log(window)
console.log(this === window)

function noArgs() {
    console.log('arguments obj from noArgs: ', arguments)
}

noArgs()

function showArgs(arg1, arg2) {
    console.log("arguments: ", arguments)
    return `arg1: ${arg1} and arg2: ${arg2}`
}

showArgs('hello', 'world')
```

```js
'use strict';

function globalExample() {
  console.log("Global `this`:", this); // undefined in strict mode
}

function counterFactory() {
  let count = 0; // part of this execution context

  return function counter() {
    count++;
    console.log(`Count: ${count}`);
  };
}

function hoistingDemo() {
  console.log(msg); // undefined due to var hoisting
  var msg = "Hoisted!";
  
  try {
    console.log(value); // ReferenceError due to TDZ
    let value = 10;
  } catch (e) {
    console.error("TDZ Error:", e.message);
  }
}

function recursive(depth) {
  if (depth === 0) return;
  console.log("Depth:", depth);
  recursive(depth - 1); // New context each time
}

globalExample(); // Execution context 1
const counter = counterFactory(); // Execution context 2
counter(); // Uses closure
counter();

hoistingDemo();
recursive(3); // Shows stack creation
```

🧠 **How to walk through this in interview**:

- Explain hoisting with `var` and TDZ with `let`.
    
- Show how closures retain state across calls.
    
- Describe how `recursive()` builds nested execution contexts.
    
- Point out strict mode effect on `this`.

---

# 2. Lexical Environment

### What is a Lexical Environment?

A **Lexical Environment** is the structure that holds **variable/function declarations** in the **scope** where code is written (lexically).

It consists of:

- **Environment Record**: Where variables/functions are physically stored.
    
- **Outer Lexical Environment Reference**: A link to the parent environment.
    

### Lexical vs Dynamic Scope:

- **Lexical Scope (JS)**: Scope is determined at **write-time**, not at runtime.
    
- **Dynamic Scope** (not in JS): Scope is based on the **call stack** at runtime.
    

### Lexical Environment Chain:

- Forms the **scope chain**.
    
- JS engine checks the current environment first → parent → parent → … → global.
    
- Enables **variable resolution** and **closure** formation.
    

### Key Concepts:

- **Closures** are formed when inner functions retain access to variables in their outer lexical environment, even after the outer function finishes.
    
- Every time a function is called, a new **execution context** is created, and with it, a new **lexical environment**.
    
- **Block scopes** (like `if`, `for`, `try`) also create their own lexical environments for `let`/`const`.
    

---

## 2. 🎯 10 Interview Questions with Detailed Answers

---

### 1. **Theory:**

**Q:** What is the difference between Lexical Environment and Execution Context?

**A:**

- Lexical Environment holds the actual variable/function bindings and links to outer scopes.
    
- Execution Context includes the Lexical Environment + `this` binding + `arguments` object.
    

---

### 2. **Code Understanding:**

**Q:** What is the output?

```js
let x = 10;
function outer() {
  let x = 20;
  function inner() {
    console.log(x);
  }
  inner();
}
outer();
```

**A:**  
`20`  
Because `inner()` closes over the **lexical** scope of `outer()` — not the global one.

---

### 3. **Code Edge Case:**

**Q:** Explain the output:

```js
let x = 1;
function test() {
  console.log(x);
  let x = 2;
}
test();
```

**A:**  
`ReferenceError: Cannot access 'x' before initialization`  
Due to the **TDZ** in the block-level lexical environment — `let x` is hoisted but uninitialized.

---

### 4. **Closures and Lexical Scope:**

**Q:** Why does this work?

```js
function makeAdder(x) {
  return function(y) {
    return x + y;
  };
}
const add5 = makeAdder(5);
console.log(add5(3));
```

**A:**  
`add5` retains access to `x` from its outer lexical environment even after `makeAdder` has finished execution — this is closure.

---

### 5. **Nested Functions:**

**Q:** Explain the scope chain here:

```js
function a() {
  let name = "outer";
  function b() {
    let title = "inner";
    console.log(name); // ?
  }
  b();
}
a();
```

**A:**  
`b()` has access to `name` because it’s defined **lexically inside** `a()`, so the JS engine traverses up the lexical environment chain to find `name`.

---

### 6. **Real-World Bug:**

**Q:** What’s wrong with this?

```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
```

**A:**  
Prints `3` three times.  
`var` is function-scoped, so all closures reference the **same** `i` in the same lexical environment. Use `let` to create new lexical environment per iteration.

---

### 7. **Debugging:**

**Q:** How do DevTools help visualize lexical environments?

**A:**

- Use the Sources tab.
    
- Step through with breakpoints.
    
- Use **Scope** section to see current Lexical Environment and parent closures.
    

---

### 8. **Scope Chain Debugging:**

**Q:** Why doesn’t this print "Hello"?

```js
function outer() {
  function inner() {
    console.log(msg);
  }
  inner();
}
outer();
let msg = "Hello";
```

**A:**  
`msg` is declared **after** `outer` runs, so not available lexically to `inner`. The JS engine checks `inner` → `outer` → global — but `msg` is in TDZ at that point.

---

### 9. **Arrow Functions + Lexical `this`:**

**Q:** How is `this` related to lexical environments in arrow functions?

**A:**  
Arrow functions **lexically bind `this`** — they inherit `this` from the enclosing lexical environment, unlike regular functions that create their own `this`.

---

### 10. **Best Practices:**

**Q:** How can understanding lexical environments help prevent bugs?

**A:**

- Avoid relying on global variables.
    
- Use `let`/`const` for block-level lexical environments.
    
- Understand closure memory to avoid leaks.
    
- Know what variables your closures might capture unintentionally.
    

---

## 3. 🧠 Micro Notes (Quick Revision)

- Lexical Environment = scope + variable bindings.
    
- Created when code is **defined**, not called.
    
- Includes **Environment Record** and link to outer.
    
- Forms the **scope chain** for variable lookup.
    
- Inner functions close over their outer lexical environments.
    
- Closures use the LE to retain state.
    
- `let/const` are hoisted to block-level LEs (with TDZ).
    
- Arrow functions inherit `this` lexically.
    
- Loops with `let` create new LEs per iteration.
    
- Use DevTools Scope tab to inspect LEs.
    

---

## 4. 💡 Demonstration Code Snippet

```js
'use strict';

let globalVar = '🌍';

function createUser(name) {
  let userType = 'admin'; // Outer lexical environment

  return function logUser() {
    let time = new Date().toISOString(); // Inner lexical env
    console.log(`[${time}] ${name} is ${userType}, Global: ${globalVar}`);
  };
}

const logger = createUser('Vamsi'); // Closure created
logger(); // Uses outer lexical environment

// Loop demo with block scope
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(`Loop i: ${i}`), 0);
}

// Arrow function lexical `this`
const obj = {
  name: 'Lexical',
  regular: function () {
    console.log("Regular:", this.name); // Lexical
  },
  arrow: () => {
    console.log("Arrow:", this.name); // `this` from global
  },
};
obj.regular();
obj.arrow();
```

🧠 **How to walk through this in interview:**

- Show how `logger()` retains access to `name` and `userType`.
    
- Show `for(let i...)` works due to fresh LE per iteration.
    
- Explain `arrow` vs `regular` function in context of lexical `this`.
    

---