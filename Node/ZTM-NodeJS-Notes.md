# Table-of-Contents

<!-- toc -->

- [Section-3-Node-Funcdamentals-Internals](#section-3-node-funcdamentals-internals)
    + [Is Node Multi-Threadead?](#is-node-multi-threadead)
    + [The Event Loop](#the-event-loop)
    + [Callback Queues](#callback-queues)
    + [Phases of the Event Loop](#phases-of-the-event-loop)
    + [What Is Node.js Best At?](#what-is-nodejs-best-at)
    + [The Node Event Emitter](#the-node-event-emitter)
- [Section 4: Node.js Fundamentals: Module System](#section-4-nodejs-fundamentals-module-system)
    + [The `require` function](#the-require-function)
    + [Making HTTP Requests](#making-http-requests)
    + [Why use modules?](#why-use-modules)
    + [Creating Our Own Modules](#creating-our-own-modules)
    + [Exporting from Modules](#exporting-from-modules)
    + [Common JS vs ECMASc](#common-js-vs-ecmasc)

<!-- tocstop -->

---

# Section-3-Node-Funcdamentals-Internals
### Is Node Multi-Threadead?
  
- JS is single-threaded; Node runs on one main thread (V8 + Node APIs + libuv event loop).
- Blocking functions halt execution until done.
- libuv handles async I/O: File System + Network operations.
- Event loop delegates async tasks to OS kernel or libuv thread pool.
- Network ops → OS kernel (avoids thread pool).
- File System ops → thread pool (default 4 threads).
- Thread pool threads run tasks in parallel, reused to save CPU.
- If thread pool is full, tasks wait until a thread is free.
- When tasks complete, event loop gets result & executes callback.
- Node hides threading complexity, enabling simple async, non-blocking code.
### The Event Loop

- **Event loop** = core of Node runtime; handles all callbacks, enabling async execution in single-threaded JS.
- Runs inside **libuv** (C language), not JavaScript.
- On program start → event loop begins (a while loop in C).
- Loop continues until Node process should exit (`should_exit` flag).
- Each iteration:
    1. Check if process should exit.
    2. If not, **process events** (execute pending callbacks).
    3. If no events, wait until one is triggered.
    4. Delegate heavy work to OS or thread pool when needed.
- After processing all current events → loop restarts.
- Stops only when no code/events remain to run.

### Callback Queues

1. Event loop (in libuv, C code) processes async events and their callbacks in a loop.
2. Node enters the event loop automatically at program start, exits when no work remains.
3. Async functions (e.g., `setTimeout`, FS read) run in OS or thread pool.
4. When operation finishes → callback placed in **callback queue**.
5. Queue = **FIFO** (First-In, First-Out): oldest callback runs first.
6. New callbacks added at queue’s bottom; processed from top.
7. Ensures fairness—callbacks run without interrupting each other.
8. **Callback queue**, **event queue**, and **message queue** are interchangeable terms.

### Phases of the Event Loop

1. Event loop (libuv) has **multiple queues**, not one.
2. **4 main phases** for JS callbacks:
    1. **Timers** → `setTimeout`, `setInterval`.
    2. **I/O callbacks** → network, file ops, general async ops.
    3. **Set Immediate** → runs after I/O phase, before next loop tick.
    4. **Close callbacks** → cleanup after closing files/connections.
3. Each phase has its own queue (FIFO order).
4. **Set Immediate** ≠ instant; executes _after_ I/O callbacks.
5. Loop order: Timers → I/O → Set Immediate → Close → repeat.
6. Other phases (`idle`, `prepare`) exist but are internal to Node.
7. Purpose: ensures all async callbacks run in a fair, organized sequence.

### What Is Node.js Best At?

1. **Best suited for**: I/O-heavy, network-heavy, real-time applications.
2. **Strength**: Delegates I/O work to OS & hardware while keeping CPU free for coordination.
3. **Not ideal for**: CPU-intensive tasks (e.g., video processing, machine learning).
4. **Why**: Heavy CPU usage blocks the event loop, harming parallel task management.
5. **Examples of good fits**:
    - Web servers
    - APIs
    - Database coordination
    - Real-time chat apps
    - Video streaming services (e.g., Netflix uses Node.js)
6. **Core role**: Acts as "glue" connecting services, databases, and APIs in modern web apps.
7. **Philosophy**: Made for the modern web’s service-based architecture.
### The Node Event Emitter

- **EventEmitter** (from `events` module) implements the **Observer Pattern** in Node.

```javascript
	const EventEmitter = require('events');
	const celeb = new EventEmitter();
```

- **Listening**: `emitter.on(eventName, callback)` → Registers a listener (callback) for `eventName`. Multiple listeners allowed.
- **Emitting**: `emitter.emit(eventName, ...args)` → Triggers all listeners for `eventName`, passing optional args.
- Real-world: `process` is an EventEmitter (e.g., `process.on('exit', code => {...})`).
- Can pass arguments to listeners for conditional handling.
- Useful for async, decoupled communication between parts of an app.

# Section 4: Node.js Fundamentals: Module System
### The `require` function
- **Purpose**: Reuse code instead of building everything from scratch; focus on unique app features.
- **Types of Modules**:
    1. **Built-in Modules** → Provided by Node (e.g., `http`, `fs`, `events`).
    2. **Local Modules** → Files/modules you create.
    3. **Third-Party Modules** → Shared by others via npm.
- **`require()`**:
    - Node-specific function (not in vanilla JS).
    - Syntax:
        ```js
        const moduleName = require('module');
        ```
    - Loads module → executes it → returns its exported content.
    - For **local files**:
        ```js
        const myModule = require('./myModule.js');
        ```
- **Best Practices**:
    - Store required module in a `const`.
    - Organize code into small, manageable modules for clarity and reuse.
- **Benefits**:
    - Code reusability.
    - Separation of concerns.
    - Easier maintenance & testing.
### Making HTTP Requests

- **Built-in Modules**: No npm; e.g., `http`, `https`, `fs`, `crypto`.
- **Purpose**: Make HTTP/HTTPS requests.
- **Import**:
    ```js
    const { request, get } = require('https');
    ```
    
- **`request(url, cb)`**:
    - `res` = EventEmitter → `'data'`, `'end'`, `'error'`.    
    - Must call `.end()`.
        
    ```js
    const req = request('https://ex.com', r=>{
      r.on('data', c=>console.log(c));
      r.on('end', ()=>console.log('done'));
    }); req.end();
    ```
- **`get(url, cb)`**: GET-only shortcut, auto `.end()`.
- **http vs https**: plain vs TLS/SSL encrypted; match module to UR
- **Best Practices**:
    
    - Match protocol.
    - Handle `'error'`.
    - Modularize logic.
    - Use destructuring for clarity.
- **Interview Hot Points**:
    
    - Diff `http`/`https`.
    - EventEmitter in `res`.
    - `.end()` in `request` vs `get`.

### Why use modules?
- **Module** = Self-contained “box” of code for one task.
    
- **Goal**: Combine modules → build complex programs.
    
- **3 Benefits**:
    
    1. **Reuse** proven code (don’t reinvent).
        
    2. **Organize** code into logical parts.
        
    3. **Encapsulation** → expose only what’s needed, hide internal details.
        
- **Example**: `http` module split into:
    
    - **Request object** → packages & sends data.
        
    - **Response object** → stores & processes server data.
        
- Higher-level modules (e.g., FTP) can use Request & Response without knowing implementation details.
    
- **Encapsulation advantage**:
    
    - Simplifies top-level logic.
        
    - Changes inside a module don’t break consumers (as long as public API stays same).

### Creating Our Own Modules
### Exporting from Modules
### Common JS vs ECMASc