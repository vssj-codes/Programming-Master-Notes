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
- [3. Hoisting](#3-hoisting)
    + [What is Hoisting?](#what-is-hoisting)
    + [Key Rules:](#key-rules)
    + [Behind the Scenes:](#behind-the-scenes)
  * [2. 🎯 10 Interview Questions with Detailed Answers](#2-%F0%9F%8E%AF-10-interview-questions-with-detailed-answers-2)
    + [1. **Theory:**](#1-theory-2)
    + [2. **Code-Based Scenario (Var Hoisting):**](#2-code-based-scenario-var-hoisting)
    + [3. **TDZ Case (Let Hoisting):**](#3-tdz-case-let-hoisting)
    + [4. **Function Hoisting vs Expression:**](#4-function-hoisting-vs-expression)
    + [5. **Edge Case: Overriding Function with Var**](#5-edge-case-overriding-function-with-var)
    + [6. **Real-World Bug:**](#6-real-world-bug-1)
    + [7. **Best Practice Question:**](#7-best-practice-question)
    + [8. **Class Hoisting:**](#8-class-hoisting)
    + [9. **Function Inside Block:**](#9-function-inside-block)
    + [10. **Debugging Hoisting:**](#10-debugging-hoisting)
  * [3. 🧠 Micro Notes (Quick Revision)](#3-%F0%9F%A7%A0-micro-notes-quick-revision-2)
  * [4. 💡 Demonstration Code Snippet (Simple + Interview-Ready)](#4-%F0%9F%92%A1-demonstration-code-snippet-simple--interview-ready)
    + [🗣️ Interview Walkthrough:](#%F0%9F%97%A3%EF%B8%8F-interview-walkthrough)
- [4. Scopes](#4-scopes)
    + [What is Scope?](#what-is-scope)
    + [Types of Scope in JavaScript:](#types-of-scope-in-javascript)
    + [Scope Chain:](#scope-chain)
    + [Closures:](#closures)
    + [`var` vs `let`/`const`:](#var-vs-letconst)
    + [Shadowing:](#shadowing)
  * [2. 🎯 10 Interview Questions with Detailed Answers](#2-%F0%9F%8E%AF-10-interview-questions-with-detailed-answers-3)
    + [1. **Theory:**](#1-theory-3)
    + [2. **Code Scenario:**](#2-code-scenario)
    + [3. **Shadowing:**](#3-shadowing)
    + [4. **Closures + Scope:**](#4-closures--scope)
    + [5. **Loop + Scope Trap:**](#5-loop--scope-trap)
    + [6. **Global Pollution:**](#6-global-pollution)
    + [7. **Best Practices:**](#7-best-practices)
    + [8. **Scope Chain Debugging:**](#8-scope-chain-debugging-1)
    + [9. **Function Declaration Inside Block:**](#9-function-declaration-inside-block)
    + [10. **TDZ (Temporal Dead Zone):**](#10-tdz-temporal-dead-zone)
  * [3. 🧠 Micro Notes (Quick Revision)](#3-%F0%9F%A7%A0-micro-notes-quick-revision-3)
  * [4. 💡 Demonstration Code Snippet (Planet Analogy 🌍)](#4-%F0%9F%92%A1-demonstration-code-snippet-planet-analogy-%F0%9F%8C%8D)
    + [🗣️ How to explain this in an interview:](#%F0%9F%97%A3%EF%B8%8F-how-to-explain-this-in-an-interview)
- [5. Scope Chain](#5-scope-chain)
  * [1. Notes](#1-notes)
    + [What is a Scope Chain?](#what-is-a-scope-chain)
    + [Analogy:](#analogy)
    + [How it's Formed:](#how-its-formed)
    + [Key Rules:](#key-rules-1)
  * [2. 🎯 10 Interview Questions with Detailed Answers](#2-%F0%9F%8E%AF-10-interview-questions-with-detailed-answers-4)
    + [1. **Theory:**](#1-theory-4)
    + [2. **Code Understanding:**](#2-code-understanding-1)
    + [3. **ReferenceError Case:**](#3-referenceerror-case)
    + [4. **Variable Hiding (Shadowing via Scope Chain):**](#4-variable-hiding-shadowing-via-scope-chain)
    + [5. **Real-World Bug:**](#5-real-world-bug-1)
    + [6. **Closures + Scope Chain:**](#6-closures--scope-chain)
    + [7. **Nested Functions Accessing Outer Scopes:**](#7-nested-functions-accessing-outer-scopes)
    + [8. **Debugging Scope Chain:**](#8-debugging-scope-chain)
    + [9. **Best Practice:**](#9-best-practice)
    + [10. **Scope Chain ≠ Call Stack:**](#10-scope-chain-%E2%89%A0-call-stack)
  * [3. 🧠 Micro Notes (Quick Revision)](#3-%F0%9F%A7%A0-micro-notes-quick-revision-4)
  * [4. 💡 Demonstration Code Snippet (Space Analogy 🚀)](#4-%F0%9F%92%A1-demonstration-code-snippet-space-analogy-%F0%9F%9A%80)
    + [🗣️ Interview Explanation:](#%F0%9F%97%A3%EF%B8%8F-interview-explanation)
- [6. [/[/ scope /]/]](#6--scope-)
- [7. Data types- Primitive](#7-data-types--primitive)
  * [**1. Strong Conceptual Notes (Primitive Data Types)**](#1-strong-conceptual-notes-primitive-data-types)
  * [**2. 10 Challenging Interview-Style Questions with Answers**](#2-10-challenging-interview-style-questions-with-answers)
  * [**3. Micro Notes (Quick Revision)**](#3-micro-notes-quick-revision)
  * [**4. Demonstration Code Snippet (Interview-Ready)**](#4-demonstration-code-snippet-interview-ready)

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
# 3. Hoisting

### What is Hoisting?

**Hoisting** is JavaScript's default behavior of moving **declarations** to the **top of their scope** during the **creation phase** of execution.

### Key Rules:

- **Only declarations are hoisted**, not initializations.
    
- **`var` is hoisted** and initialized as `undefined`.
    
- **`let` and `const` are hoisted**, but remain **uninitialized** in the **Temporal Dead Zone (TDZ)**.
    
- **Function declarations** are fully hoisted (code + definition).
    
- **Function expressions** are treated like variables.
    

### Behind the Scenes:

During the creation phase:

- JS engine sets up memory space for variables and functions.
    
- `var` → `undefined`
    
- `let`/`const` → TDZ (can’t be accessed until line of declaration)
    
- Function declarations → Hoisted completely (name + body)
    

---

## 2. 🎯 10 Interview Questions with Detailed Answers

---

### 1. **Theory:**

**Q:** What gets hoisted in JavaScript?

**A:**

- All **declarations** (`var`, `let`, `const`, `function`, `class`) are hoisted.
    
- Only `var` and `function` declarations are usable before their line.
    
- `let`, `const`, and `class` stay in TDZ and throw ReferenceError if accessed early.
    

---

### 2. **Code-Based Scenario (Var Hoisting):**

```js
console.log(a);
var a = 10;
```

**A:**  
Output: `undefined`  
`a` is hoisted and initialized to `undefined`. The assignment `10` happens later.

---

### 3. **TDZ Case (Let Hoisting):**

```js
console.log(b);
let b = 20;
```

**A:**  
Throws: `ReferenceError: Cannot access 'b' before initialization`  
`b` is hoisted but remains in TDZ until the `let` line executes.

---

### 4. **Function Hoisting vs Expression:**

```js
greet(); // Works
function greet() {
  console.log('Hello');
}

wave(); // Error
var wave = function () {
  console.log('Hi');
};
```

**A:**

- `greet` is hoisted as a complete function.
    
- `wave` is hoisted as a variable (`undefined`), not as a function.
    

---

### 5. **Edge Case: Overriding Function with Var**

```js
console.log(typeof sayHi);
var sayHi = "👋";
function sayHi() {
  console.log("Hello");
}
```

**A:**  
Output: `string`  
Function `sayHi` is hoisted first, but then `var sayHi` re-declares it (overwrites it) with `undefined`, then later it becomes a string.

---

### 6. **Real-World Bug:**

```js
function init() {
  console.log(config);
  var config = "API_KEY";
}
init();
```

**A:**  
Logs `undefined`, not an error.  
This can lead to **silent bugs** where you expect `config` to exist, but get `undefined` due to hoisting.

---

### 7. **Best Practice Question:**

**Q:** How to avoid hoisting pitfalls?

**A:**

- Use `let`/`const` over `var`.
    
- Declare all variables at the top of their scope.
    
- Avoid function expressions before definition.
    
- Use linters like ESLint to catch hoisting-related bugs.
    

---

### 8. **Class Hoisting:**

```js
const user = new Person();
class Person {}
```

**A:**  
Throws: `ReferenceError: Cannot access 'Person' before initialization`  
`class` declarations are hoisted but in TDZ, just like `let`/`const`.

---

### 9. **Function Inside Block:**

```js
if (true) {
  function say() {
    console.log("Hi");
  }
}
say();
```

**A:**  
In **strict mode**, this throws an error (`say is not defined`) because function declarations inside blocks are scoped to that block.

---

### 10. **Debugging Hoisting:**

**Q:** How to debug a ReferenceError caused by hoisting?

**A:**

- Check if you’re accessing `let`/`const`/`class` before declaration.
    
- Look for TDZ issues in the call stack.
    
- Use breakpoints and scope inspector in DevTools to see what’s initialized.
    

---

## 3. 🧠 Micro Notes (Quick Revision)

- **Hoisting = declarations move to top** (during creation phase).
    
- `var` → hoisted, initialized as `undefined`.
    
- `let`/`const`/`class` → hoisted but in **TDZ**.
    
- Function **declarations** → fully hoisted.
    
- Function **expressions** → only the variable part is hoisted.
    
- TDZ = accessing `let`/`const` before they’re initialized throws error.
    
- `class` behaves like `const` (TDZ until initialized).
    
- Use `let`/`const`, never rely on `var`.
    
- Watch out for hidden bugs due to hoisted `undefined`.
    

---

## 4. 💡 Demonstration Code Snippet (Simple + Interview-Ready)

```js
'use strict';

console.log("1. varDemo:", varDemo); // undefined
var varDemo = "I'm hoisted";

try {
  console.log("2. letDemo:", letDemo); // ReferenceError
  let letDemo = "I'm in TDZ";
} catch (err) {
  console.error("2. letDemo Error:", err.message);
}

greet(); // Works due to full hoisting
function greet() {
  console.log("3. greet(): Function declaration hoisted");
}

try {
  wave(); // ReferenceError
  var wave = function () {
    console.log("This won't run");
  };
} catch (err) {
  console.error("4. wave Error:", err.message);
}

// Class hoisting test
try {
  const obj = new Planet();
  class Planet {}
} catch (err) {
  console.error("5. class hoisting Error:", err.message);
}
```

---

### 🗣️ Interview Walkthrough:

> “Here, `varDemo` is hoisted and initialized with `undefined`. But `letDemo` is hoisted but stuck in TDZ, causing a ReferenceError. The `greet` function is fully hoisted, so it works. `wave`, on the other hand, is hoisted only as a variable. Same with the class — it's hoisted like `let`, and accessing it before declaration causes a ReferenceError.”

---
# 4. Scopes

### What is Scope?

**Scope** determines the **visibility** and **lifetime** of variables, functions, and objects in a program — i.e., where you can access them.

---

### Types of Scope in JavaScript:

|Scope Type|Description|
|---|---|
|**Global**|Accessible everywhere. Created outside all functions.|
|**Function**|Created inside a function; `var` is function-scoped.|
|**Block**|Created inside `{ }`; `let` and `const` are block-scoped.|
|**Module**|ES6 modules have their own top-level scope (not global).|
|**Lexical**|Scope defined by code position, not call stack (a.k.a. static scope).|

---

### Scope Chain:

- When accessing a variable, JS looks **locally** first, then checks **parent scopes** recursively until global is reached.
    
- If not found, → `ReferenceError`.
    

---

### Closures:

- Inner functions **remember** variables from outer scopes via lexical scoping — this is how closures work.
    

---

### `var` vs `let`/`const`:

- `var` is **function scoped**, ignores block `{}`.
    
- `let`/`const` are **block scoped**, respect `{}`.
    

---

### Shadowing:

- When an inner scope declares a variable with the same name as an outer scope — it "shadows" the outer one within that block.
    

---

## 2. 🎯 10 Interview Questions with Detailed Answers

---

### 1. **Theory:**

**Q:** What's the difference between function scope and block scope?

**A:**

- **Function scope** (e.g., `var`) means the variable is only accessible within the entire function, not within a block.
    
- **Block scope** (e.g., `let`, `const`) limits access to the specific `{}` block (like loops or if statements).
    

---

### 2. **Code Scenario:**

```js
if (true) {
  var a = 1;
  let b = 2;
}
console.log(a); // ?
console.log(b); // ?
```

**A:**  
`a = 1` (accessible — function scoped)  
`b = ReferenceError` (block scoped)

---

### 3. **Shadowing:**

```js
let x = 10;
function test() {
  let x = 5;
  console.log(x);
}
test();
```

**A:**  
Output: `5` — the `x` inside `test()` shadows the outer `x`.

---

### 4. **Closures + Scope:**

```js
function outer() {
  let secret = '🔐';
  return function inner() {
    return secret;
  };
}
const fn = outer();
console.log(fn());
```

**A:**  
Output: `🔐` — `inner()` has access to `outer()`’s scope even after `outer()` returns.

---

### 5. **Loop + Scope Trap:**

```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
```

**A:**  
Output: `3 3 3` — all closures refer to the same `i` (function scope).  
To fix: use `let i` for block scoping.

---

### 6. **Global Pollution:**

**Q:** What’s the risk of using `var` globally?

**A:**  
It attaches variables to `window` (in browsers), potentially **overwriting** existing properties or libraries and leading to hard-to-debug issues.

---

### 7. **Best Practices:**

**Q:** What are some scope-related best practices?

**A:**

- Prefer `const` for immutables, `let` for block-scoped vars.
    
- Avoid using `var`.
    
- Declare variables at the top of their scope.
    
- Keep scopes shallow and small for easier debugging.
    

---

### 8. **Scope Chain Debugging:**

```js
function one() {
  let a = "A";
  function two() {
    let b = "B";
    function three() {
      console.log(a, b);
    }
    three();
  }
  two();
}
one();
```

**A:**  
Logs: `A B` — `three()` has access to all ancestor scopes due to lexical scope.

---

### 9. **Function Declaration Inside Block:**

```js
if (true) {
  function sayHi() {
    return 'Hi';
  }
}
console.log(sayHi());
```

**A:**  
In **strict mode**, this throws a `ReferenceError`. Function declarations inside blocks are block-scoped per ES6.

---

### 10. **TDZ (Temporal Dead Zone):**

```js
{
  console.log(a);
  let a = 2;
}
```

**A:**  
`ReferenceError` — `a` is block-scoped and in the **TDZ** until declared.

---

## 3. 🧠 Micro Notes (Quick Revision)

- **Scope = visibility of variables.**
    
- `var` → function scoped.
    
- `let/const` → block scoped.
    
- **Lexical scope** = defined by where functions are written.
    
- Scope chain: inner → outer → global.
    
- **Shadowing** = inner var hides outer var.
    
- **Closures** use outer scopes even after function exits.
    
- **Avoid global scope pollution.**
    
- **TDZ** → error if `let/const` used before declaration.
    

---

## 4. 💡 Demonstration Code Snippet (Planet Analogy 🌍)

```js
'use strict';

let galaxy = '🌌 Milky Way'; // Global scope

function createPlanet(planet) {
  let atmosphere = 'Oxygen'; // Function scope

  if (planet === 'Earth') {
    let life = true; // Block scope
    console.log(`Inside block: ${planet} has life: ${life}`);
  }

  // console.log(life); // ReferenceError: block scoped

  return function describe() {
    console.log(`${planet} has ${atmosphere} in the ${galaxy}`);
  };
}

const earthInfo = createPlanet('🌍 Earth');
earthInfo(); // Uses all 3 scopes
```

---

### 🗣️ How to explain this in an interview:

> "Here, we have variables in global, function, and block scope. The `describe()` function uses lexical scoping to access them all. The `life` variable is block scoped, so it’s inaccessible outside the `if`. This cleanly shows how different scopes coexist and how closures retain access to outer scopes."

---

# 5. Scope Chain
## 1. Notes

### What is a Scope Chain?

The **scope chain** is the chain of **lexical environments** that JavaScript uses to **resolve variable identifiers**.

When a variable is accessed:

1. JS looks in the **current scope**.
    
2. If not found, it checks the **outer (parent) scope**.
    
3. This continues up until the **global scope**.
    
4. If still not found → `ReferenceError`.
    

---

### Analogy:

Imagine asking your friend a question. If they don’t know, they ask their parent. If the parent doesn’t know, they ask a grandparent — this is the scope chain.

---

### How it's Formed:

- Every time a function is created, it remembers the **scope in which it was defined** (its lexical environment).
    
- These nested scopes create a **chain of accessible variables**.
    

---

### Key Rules:

- The scope chain is based on **where functions are defined**, **not** where they are called.
    
- Used for **variable lookup**, not `this` binding.
    

---

## 2. 🎯 10 Interview Questions with Detailed Answers

---

### 1. **Theory:**

**Q:** What is the scope chain used for?

**A:**  
To **resolve variable names** during execution. JS looks through the nested chain of lexical environments until it finds the variable or throws an error.

---

### 2. **Code Understanding:**

```js
let a = 'global';

function one() {
  let b = 'one';
  function two() {
    let c = 'two';
    console.log(a, b, c);
  }
  two();
}
one();
```

**A:**  
Logs: `global one two`  
Because `two()` can access its own scope, then `one()`’s, then global — thanks to the scope chain.

---

### 3. **ReferenceError Case:**

```js
function outer() {
  function inner() {
    console.log(x);
  }
  inner();
}
outer();
let x = 10;
```

**A:**  
Throws `ReferenceError: x is not defined`  
Because `x` is declared **after** `outer()` is executed — it's **not in the scope chain** at the time.

---

### 4. **Variable Hiding (Shadowing via Scope Chain):**

```js
let item = 'apple';

function printItem() {
  let item = 'banana';
  console.log(item);
}
printItem();
```

**A:**  
Logs: `banana` — `printItem()`'s local `item` **shadows** the global `item`.

---

### 5. **Real-World Bug:**

**Q:** Why does this log `undefined`?

```js
let config = "global";

function setup() {
  if (false) {
    var config = "scoped";
  }
  console.log(config);
}
setup();
```

**A:**  
Because `var config` is hoisted to the top of `setup()`, making the inner `config` take precedence in the scope chain → initialized as `undefined`.

---

### 6. **Closures + Scope Chain:**

```js
function counter() {
  let count = 0;
  return function () {
    count++;
    return count;
  };
}
const increment = counter();
console.log(increment());
console.log(increment());
```

**A:**  
Returns `1`, then `2`.  
The inner function forms a **closure** and keeps `count` alive via the scope chain.

---

### 7. **Nested Functions Accessing Outer Scopes:**

```js
function grandParent() {
  let gp = "👴";
  function parent() {
    let p = "👨";
    function child() {
      let c = "👶";
      console.log(gp, p, c);
    }
    child();
  }
  parent();
}
grandParent();
```

**A:**  
Logs: `👴 👨 👶` — the child function walks up the scope chain to access outer variables.

---

### 8. **Debugging Scope Chain:**

**Q:** How to inspect scope chain in DevTools?

**A:**

- Use breakpoints in the Sources tab.
    
- Look at the **"Scope"** panel — it shows Local, Closure, Script, and Global scopes in chain order.
    

---

### 9. **Best Practice:**

**Q:** How can understanding the scope chain prevent bugs?

**A:**

- Helps identify **where a variable is coming from**.
    
- Avoids accidental **global variable access**.
    
- Useful in debugging `undefined` or `ReferenceError` issues.
    

---

### 10. **Scope Chain ≠ Call Stack:**

**Q:** Is the scope chain the same as the call stack?

**A:**  
No. Scope chain is about **where functions are defined**.  
Call stack is about **which function is currently being executed**.

---

## 3. 🧠 Micro Notes (Quick Revision)

- **Scope chain = chain of parent scopes used to find variables.**
    
- JS checks current → parent → grandparent → ... → global.
    
- Based on **function definition location**, not where it's called.
    
- Used in closures, variable lookup.
    
- If not found in chain → `ReferenceError`.
    
- Useful for debugging bugs like `undefined` or shadowed vars.
    
- `let`/`const` respect block scope in the chain; `var` is function-scoped.
    

---

## 4. 💡 Demonstration Code Snippet (Space Analogy 🚀)

```js
'use strict';

let universe = '🌌 Universe'; // Global

function galaxy() {
  let starSystem = '🌟 Solar System'; // Enclosed in galaxy

  function planet() {
    let moon = '🌕 Moon'; // Enclosed in planet
    console.log(`${moon} is in ${starSystem} inside ${universe}`);
  }

  planet();
}

galaxy(); // Outputs: 🌕 is in 🌟 Solar System inside 🌌 Universe
```

---

### 🗣️ Interview Explanation:

> “When `planet()` runs, it looks for `moon` in its own scope. Then it looks for `starSystem` in the `galaxy()` scope. Then `universe` in the global scope. That’s the scope chain in action — inner functions accessing variables defined lexically in parent scopes.”

---
# 6. [/[/ scope /]/]

```javascript
// Lexical Env = [[scope]]
function a() {
}
a()
```

---
# 7. Data types- Primitive
## **1. Strong Conceptual Notes (Primitive Data Types)**

**Definition:**  
Primitive data types are immutable, single-value types in JavaScript, stored directly in memory by value (not by reference).

**7 Primitive Types in JavaScript:**

1. **Number** – Both integer and floating-point, IEEE 754 double precision.
    
2. **String** – Sequence of UTF-16 code units.
    
3. **Boolean** – `true` or `false`.
    
4. **Null** – Intentional absence of any value (`typeof null` is `"object"` — historical bug).
    
5. **Undefined** – Declared but not assigned.
    
6. **Symbol** – Unique and immutable, used as object property keys.
    
7. **BigInt** – For arbitrarily large integers beyond `Number.MAX_SAFE_INTEGER`.
    

**Key Characteristics:**

- **Immutable:** Any change creates a new value.
    
- **Pass-by-value:** Copy is passed, not the original.
    
- **Stored in stack** (small, fixed-size values).
    
- **Type Coercion:** Automatic conversion when needed (can cause bugs).
    

**Special Cases:**

- `NaN` is a **Number** (`typeof NaN === "number"`).
    
- `+0` and `-0` are distinct but mostly behave the same.
    
- `Symbol` and `BigInt` are ES6+ and ES2020 additions.
    
- String methods return new strings, do not mutate original.
    

**When to Use:**

- Use primitive for fixed, small data values.
    
- Use `Symbol` for object property keys that must be unique.
    
- Use `BigInt` for exact large integer math (like IDs, crypto).
    

---

## **2. 10 Challenging Interview-Style Questions with Answers**

---

**Q1.** _Why is `typeof null` `"object"` and not `"null"`?_  
**A:** It’s a legacy bug from the first JS implementation in 1995 where values were stored as type tags in binary. Objects had the tag `000`, and `null` was represented as a null pointer (`000`). Changing it now would break old code.

---

**Q2.** _What’s the difference between `undefined` and `null`?_  
**A:**

- `undefined`: A variable has been declared but not assigned.
    
- `null`: An intentional absence of value set by the developer.  
    Example:
    

```js
let a;
console.log(a); // undefined
let b = null;
console.log(b); // null
```

---

**Q3.** _What is `NaN` and how do you check for it reliably?_  
**A:**  
`NaN` stands for "Not-a-Number" but is a `number` type.

- Fails equality checks: `NaN === NaN` → false
    
- Check: `Number.isNaN(value)` (better than global `isNaN` which coerces).  
    Example:
    

```js
Number.isNaN("abc"); // false
isNaN("abc"); // true (bad)
```

---

**Q4.** _What’s the difference between `Symbol()` and `Symbol.for()`?_  
**A:**

- `Symbol()` creates a new unique symbol every time.
    
- `Symbol.for()` checks a global registry; if symbol exists, returns it; else creates it.
    

```js
Symbol("x") === Symbol("x"); // false
Symbol.for("x") === Symbol.for("x"); // true
```

---

**Q5.** _What’s the issue with adding `Number` and `BigInt`?_  
**A:**  
They cannot be mixed directly:

```js
1n + 1; // TypeError
```

Must convert explicitly:

```js
BigInt(1) + 1n; // works
```

---

**Q6.** _Why are primitive values immutable, and what happens when you "change" them?_  
**A:**  
You can’t alter the actual value; instead, a new value is created and the reference is updated.

```js
let str = "Hi";
str[0] = "h"; // no effect
str = "hi"; // new string created
```

---

**Q7.** _How do `==` and `===` behave with primitives?_  
**A:**

- `===`: No coercion, checks value & type.
    
- `==`: Allows coercion (e.g., `"5" == 5` → true).  
    Best practice: Always use `===`.
    

---

**Q8.** _What’s the difference between `0`, `-0`, and `Object.is`?_  
**A:**

- `0 === -0` → true
    
- `Object.is(0, -0)` → false
    
- Used in math: `1 / 0` → Infinity, `1 / -0` → -Infinity.
    

---

**Q9.** _What’s a real-world use of `Symbol` in large codebases?_  
**A:**  
To define unique keys in objects to avoid accidental overwrites in libraries/plugins:

```js
const id = Symbol("id");
user[id] = 123; // can't clash with other props
```

---

**Q10.** _How does `typeof` behave differently for primitives and objects?_  
**A:**

- Primitives: `"number"`, `"string"`, `"boolean"`, `"undefined"`, `"symbol"`, `"bigint"`.
    
- Objects: `"object"` (including `null`), functions: `"function"`.
    

---

## **3. Micro Notes (Quick Revision)**

- **Primitive Types:** Number, String, Boolean, Null, Undefined, Symbol, BigInt.
    
- Immutable, pass-by-value, stored in stack.
    
- `typeof null` → "object" (legacy bug).
    
- `NaN` is a number; use `Number.isNaN()` to check.
    
- `Symbol()` → unique every time; `Symbol.for()` → global registry.
    
- Cannot mix BigInt and Number directly.
    
- `===` strict, `==` allows coercion.
    
- `Object.is` detects `+0` vs `-0` and `NaN` equality.
    
- Strings immutable — all string ops return new strings.
    
- Use `Symbol` for unique keys, `BigInt` for large integers.
    

---

## **4. Demonstration Code Snippet (Interview-Ready)**

This example packs in all core primitive concepts in one story-like snippet:

```js
// Primitive Data Types Demo: Bank Account Example

// Account details
let accountNumber = 9876543210123456n; // BigInt for large ID
let holderName = "Vamsi"; // String (immutable)
let isActive = true; // Boolean
let lastTransaction = null; // Null = intentionally no transaction
let balance; // Undefined = not set yet
const accountSecret = Symbol("accountSecret"); // Unique key

let account = {
  [accountSecret]: "hashed_secret_key",
};

// Assign balance later
balance = 1200.50; // Number

// Demonstrate immutability
let oldName = holderName;
holderName = "Vamsi Krishna"; // New string, oldName unchanged

// NaN check example
let withdrawal = "abc"; 
if (Number.isNaN(Number(withdrawal))) {
  console.log("Invalid withdrawal amount");
}

// Symbol uniqueness
console.log(Symbol("x") === Symbol("x")); // false
console.log(Symbol.for("y") === Symbol.for("y")); // true

// BigInt + Number mix error (intentional)
try {
  console.log(accountNumber + 1);
} catch (e) {
  console.log("Error mixing BigInt and Number");
}

// 0 vs -0 detection
console.log(0 === -0); // true
console.log(Object.is(0, -0)); // false

// Show typeof results
console.log(typeof balance); // "number"
console.log(typeof accountSecret); // "symbol"
console.log(typeof null); // "object" (legacy bug)

```

**Interview Walkthrough Points:**

- Chose **BigInt** for account numbers beyond safe integer limit.
    
- Showed **immutability** of strings with `oldName`.
    
- Demonstrated **`NaN` safe check** with `Number.isNaN()`.
    
- Used **Symbols** for secure property keys.
    
- Exposed **BigInt + Number mixing** TypeError.
    
- Showed **`Object.is`** to detect `+0` vs `-0`.
    
- Covered **`typeof` quirks** like `null`.
    

---
