# Table-of-Contents

<!-- toc -->

- [Section-3-Node-Funcdamentals-Internals](#section-3-node-funcdamentals-internals)
    + [Is Node Multi-Threadead?](#is-node-multi-threadead)
    + [The Event Loop](#the-event-loop)

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
-