# Table-of-Contents

<!-- toc -->

- [1. 🔍 Strong Conceptual Notes](#1-%F0%9F%94%8D-strong-conceptual-notes)
  * [What is an Execution Context?](#what-is-an-execution-context)
  * [Types of Execution Context:](#types-of-execution-context)
  * [Execution Context Phases:](#execution-context-phases)
  * [Call Stack (Execution Stack):](#call-stack-execution-stack)
  * [Key Concepts:](#key-concepts)
- [2. 🎯 10 Interview Questions with Detailed Answers](#2-%F0%9F%8E%AF-10-interview-questions-with-detailed-answers)
  * [1. **Theory:**](#1-theory)
  * [2. **Code-Based Scenario:**](#2-code-based-scenario)
  * [3. **Code-Based Scenario (TDZ):**](#3-code-based-scenario-tdz)
  * [4. **Edge Case:**](#4-edge-case)
  * [5. **Real-World Bug:**](#5-real-world-bug)
  * [6. **Best Practices:**](#6-best-practices)
  * [7. **Debugging:**](#7-debugging)
  * [8. **Code Understanding:**](#8-code-understanding)
  * [9. **Closures + Execution Context:**](#9-closures--execution-context)
  * [10. **Memory Leak Case:**](#10-memory-leak-case)
- [3. 🧠 Micro Notes (Quick Revision)](#3-%F0%9F%A7%A0-micro-notes-quick-revision)
- [4. 💡 Demonstration Code Snippet](#4-%F0%9F%92%A1-demonstration-code-snippet)

<!-- tocstop -->

---
## 1. 🔍 Strong Conceptual Notes 

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
