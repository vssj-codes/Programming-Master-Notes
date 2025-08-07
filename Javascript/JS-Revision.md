# Table-of-Contents

<!-- toc -->

- [Micro notes](#micro-notes)

<!-- tocstop -->

---

1. **What is hoisting in JavaScript, and what this really means is…?**  
    **Answer:** Hoisting is the compile-time process where JavaScript “moves” declarations of variables and functions to the top of their containing scope—whether that’s global or function—before running any code. What gets moved is the declaration, not the initialization. So when you write `var x = 5`, under the hood it becomes:
    
    ```js
    var x;
    // …your code…
    x = 5;
    ```
    
    That initial `undefined` state is why you can reference a `var` before its assignment without a crash.
    
2. **Given:**
    
    ```js
    console.log(a);
    var a = 10;
    console.log(a);
    ```
    
    What logs and why?  
    **Answer:** First it logs `undefined`, then `10`. At compile time, `var a` is hoisted (initialized to `undefined`). So the first `console.log(a)` sees that undefined. Only afterward does the assignment `a = 10` run, so the second log shows `10`.
    
3. **What happens with `let`/`const`:**
    
    ```js
    console.log(b);
    let b = 20;
    ```
    
    And what’s the Temporal Dead Zone (TDZ)?  
    **Answer:** You get a `ReferenceError: Cannot access 'b' before initialization`. Here `let`/`const` are hoisted but not initialized. Between entering the block and the actual declaration you’re in the TDZ. Any access there throws, protecting you from accidental early reads.
    
4. **How do function declarations differ from function expressions in hoisting?**  
    **Answer:** Function declarations are hoisted with both name and body, so you can call them anywhere in their scope:
    
    ```js
    greet();            // works
    function greet() {} // declaration is hoisted
    ```
    
    Function expressions (including arrow functions) hoist only the variable name as `undefined`:
    
    ```js
    speak();                 // TypeError: speak is not a function
    var speak = function() {};
    ```
    
5. **In this snippet, what happens and why?**
    
    ```js
    console.log(foo);
    var foo = () => 'bar';
    console.log(foo());
    ```
    
    **Answer:** First log is `undefined` (the var is hoisted without its assignment). Then `foo()` runs fine after assignment, logging `'bar'`. Arrow functions behave like any function expression.
    
6. **Can you do `new MyClass()` before `class MyClass {}`?**  
    **Answer:** No. Class declarations are hoisted in a TDZ, similar to `let`. Attempting to instantiate before the declaration throws `ReferenceError`. The engine knows the class exists but won’t initialize it until its line.
    
7. **Analyze this and predict output:**
    
    ```js
    function test() {
      console.log(x);
      if (false) var x = 5;
      console.log(x);
    }
    test();
    ```
    
    **Answer:** Both logs are `undefined`. Inside `test`, `var x` is hoisted to top of that function (not just inside the `if`), so `x` exists from the start, initialized to `undefined`.
    
8. **Debugging angle:** Describe a scenario where hoisting caused a subtle bug in production. How would you find and fix it?  
    **Answer:** Imagine a block that conditionally declares a `var userId`, then later reuses `userId` assuming it’s from an outer scope—but ends up `undefined`. You’d spot it by noticing odd `undefined` values in logs or a stack trace pointing to unexpected scope. Enabling strict mode, running a linter rule like `no-use-before-define`, and switching to `let`/`const` often fixes it.
    
9. **How do default parameters interact with hoisting and the TDZ?**
    
    ```js
    function f(a = x, x = 5) {
      console.log(a, x);
    }
    f();
    ```
    
    **Answer:** You get `ReferenceError` for `x` when evaluating `a = x`, because default parameters are evaluated in left-to-right order in their own scope. `x` lives in the TDZ until its default is processed.
    
10. **In a modular codebase (ES modules or CommonJS), how can hoisting affect initialization order?** Give an example and mitigation.  
    **Answer:** With circular dependencies, module A might import `foo` from B before B has assigned it, yielding `undefined`. For example:
    
    ```js
    // a.js
    import { bar } from './b.js'; console.log(bar);
    export const foo = 'FOO';
    // b.js
    import { foo } from './a.js'; console.log(foo);
    export const bar = 'BAR';
    ```
    
    You’d see `undefined` first. To mitigate, refactor to avoid circular deps, or use dynamic imports so you defer evaluation until after both modules load.
    

---

#### Micro notes

- **Hoisting** lifts declarations (not assignments) to top of their scope at compile time.
    
- **var** is hoisted & initialized to `undefined`.
    
- **let/const** hoisted but uninitialized → TDZ until declaration.
    
- **Function declarations** fully hoisted (name + body).
    
- **Function expressions/arrow** only hoist the var name (as `undefined`).
    
- **Class declarations** behave like `let` (TDZ → cannot use early).
    
- **var in blocks** moves to function scope, not block.
    
- **Default params** evaluated L→R; referencing later params hits TDZ.
    
- **Debug tips:** strict mode + `no-use-before-define` lint + clear naming.
    
- **Modules:** circular imports can expose `undefined`; use dynamic imports or refactor.