# Table-of-Contents

<!-- toc -->

- [Concurrency and Parallelism](#concurrency-and-parallelism)
  * [The Core Problem: Servers Must Handle Multiple Things at Once](#the-core-problem-servers-must-handle-multiple-things-at-once)
  * [The Waste Problem: Why Concurrency Matters](#the-waste-problem-why-concurrency-matters)
    + [A Typical API Call Breakdown](#a-typical-api-call-breakdown)
  * [IO-Bound vs CPU-Bound](#io-bound-vs-cpu-bound)
  * [Concurrency vs Parallelism](#concurrency-vs-parallelism)
    + [Visual Comparison (two requests, timeline in ms)](#visual-comparison-two-requests-timeline-in-ms)
  * [How Computers Handle Multiple Things: Two Mechanisms](#how-computers-handle-multiple-things-two-mechanisms)
  * [Mechanism 1: Threads](#mechanism-1-threads)
    + [The OS Scheduler (Preemptive Scheduling)](#the-os-scheduler-preemptive-scheduling)
    + [Shared Memory Between Threads](#shared-memory-between-threads)
    + [The Cost of Threads](#the-cost-of-threads)
  * [Mechanism 2: Event Loop](#mechanism-2-event-loop)
    + [Core Principle: Never Block the Event Loop](#core-principle-never-block-the-event-loop)
    + [Callbacks → Promises → async/await](#callbacks-%E2%86%92-promises-%E2%86%92-asyncawait)
    + [async/await as a State Machine](#asyncawait-as-a-state-machine)
    + [Event Loop Efficiency](#event-loop-efficiency)
  * [Mechanism 3: Virtual Threads / Goroutines (Go's Model)](#mechanism-3-virtual-threads--goroutines-gos-model)
    + [Architecture](#architecture)
    + [How Go Handles IO](#how-go-handles-io)
    + [Go Code — No Callbacks Required](#go-code--no-callbacks-required)
  * [When to Use Each](#when-to-use-each)
  * [Concurrency Problems: Race Conditions](#concurrency-problems-race-conditions)
    + [The Lost Update Problem (Threads)](#the-lost-update-problem-threads)
    + [Race Conditions in Event Loops (async/await)](#race-conditions-in-event-loops-asyncawait)
    + [Solutions to Race Conditions](#solutions-to-race-conditions)
  * [Summary](#summary)

<!-- tocstop -->

---

# Concurrency and Parallelism

**Source:** Sriniously — Backend from First Principles (Video 23)
**Link:** [Watch](https://www.youtube.com/watch?v=estH64OkwxU)

---

## The Core Problem: Servers Must Handle Multiple Things at Once

Every backend application has one universal requirement: it must handle multiple requests simultaneously. A server that can only process one request at a time would make all other users wait or error out — completely impractical at any meaningful scale.

Understanding how operating systems, programming language runtimes, and frameworks enable multiple simultaneous operations — and the tradeoffs involved — is foundational for debugging, structuring applications, and making architectural decisions.

---

## The Waste Problem: Why Concurrency Matters

When a server processes a typical API request, it must interact with external systems (database, email provider, cache). During this interaction, the server waits for a response.

**How long does it actually wait?**

| Database location | Wait time |
|------------------|-----------|
| Same host (localhost) | 1–2ms |
| Different availability zone | 20–30ms |
| Different region | 90–100ms |

**What is the CPU doing during this wait?** Nothing. It is completely idle.

**Quantifying the waste:**

A modern CPU executes ~3 billion instructions/second = **3 million instructions/millisecond**.

If the server is waiting 100ms for a database response, it *could* have executed **300 million instructions**. Instead it executes zero.

### A Typical API Call Breakdown

A mid-complexity API call with 3–5 database queries and 1–2 external service calls (email, Redis):

| Activity | Time |
|----------|------|
| Waiting for IO (network calls, DB queries) | ~250ms |
| Actual CPU processing (business logic, parsing) | ~10ms |
| **CPU idle percentage** | **~96%** |

**This is the problem concurrency solves:** use that 96% idle time to process other requests or tasks instead of wasting it.

---

## IO-Bound vs CPU-Bound

| Type | What it means | Examples |
|------|--------------|---------|
| **IO-bound** | Work that waits for external resources; CPU is idle | Database queries, file system, network calls, logging, email APIs |
| **CPU-bound** | Work that requires active CPU computation | Parsing/validation, encryption, image processing, ML inference, video encoding |

**Most backend SaaS applications are IO-bound.** 70%+ of time is spent waiting on IO. The rare exception is applications doing heavy computation (video encoding, ML workloads, cryptography).

---

## Concurrency vs Parallelism

These are frequently confused. The distinction matters:

| | Concurrency | Parallelism |
|--|------------|------------|
| **Definition** | Dealing with multiple things at once | Doing multiple things at the same instant |
| **Hardware requirement** | Can work with a single CPU core | Requires multiple CPU cores |
| **Nature** | About program structure | About hardware execution |
| **One-liner** | "Multiple things in progress simultaneously" | "Multiple things executing at the exact same moment" |

### Visual Comparison (two requests, timeline in ms)

**Concurrency (1 core):**
```
Timeline: 0──5──────────40──50──60
Request A: [CPU]──[waiting IO]──[CPU]─done
Request B:      [CPU]──[waiting IO]──[CPU]─done
           ^ A pauses for IO → B gets CPU
```
At any given millisecond, only one request uses the CPU. But both are in progress (neither has errored or returned yet).

**Parallelism (2 cores):**
```
Timeline: 0──────────────────────60
Request A: [CPU]──[IO wait]──[CPU]─done  (core 1)
Request B: [CPU]──[IO wait]──[CPU]─done  (core 2)
           ^ Both execute at the same millisecond
```
Both requests execute simultaneously on separate cores.

> **Concurrency** = doing one thing at a time, but switching efficiently between many.
> **Parallelism** = doing multiple things at the exact same moment.

---

## How Computers Handle Multiple Things: Two Mechanisms

All concurrency primitives in every programming language (async/await in JS/Python, goroutines in Go, virtual threads in Java) are built on top of one or both of these mechanisms:

1. **Threads** (OS-level)
2. **Event loops**

---

## Mechanism 1: Threads

A **thread** is an independent unit of execution managed by the operating system.

When the OS creates a thread, it allocates:
- **Stack** — tracks function call history and local variables
- **Instruction pointer** — tracks the current execution position so it can resume after a switch
- **Data structures** — internal bookkeeping for the scheduler

### The OS Scheduler (Preemptive Scheduling)

A **scheduler** in the OS manages all threads: which runs, which pauses, which is blocked. Scheduling is **preemptive** — threads are paused whether they want to be or not, at fixed time slices (~2ms typically).

**When a thread hits IO:** It tells the OS "I'm blocked waiting for IO." The OS marks it as blocked, switches to another thread. When the IO completes, the OS marks the thread as runnable again (but it still waits its turn for CPU time).

### Shared Memory Between Threads

Threads within the same process can read/write each other's memory via pointers. This makes inter-thread communication fast (no serialization needed) but introduces dangerous shared state problems.

Processes are fully isolated from each other — a thread in process 1 cannot access memory in process 2.

### The Cost of Threads

| Overhead | Detail |
|----------|--------|
| **Memory** | Each thread's stack: ~8MB on Linux (even if mostly virtual) — 10,000 threads → gigabytes of memory |
| **Creation** | System call to OS kernel to allocate stack, data structures, add to scheduler — costs microseconds to milliseconds |
| **Context switching** | Saving/restoring CPU registers, bookkeeping, selecting next thread — 1–10µs per switch; with thousands of threads, this overhead accumulates significantly |

**The fundamental problem:** Threads-per-request does not scale. At 10,000 concurrent requests, you have 10,000 threads and gigabytes of overhead, with most time spent on context switching rather than real work.

---

## Mechanism 2: Event Loop

The event loop uses a **single thread** with a callback/queue-based model instead of multiple threads.

### Core Principle: Never Block the Event Loop

Instead of blocking and waiting for IO, a task registers a **callback** (the code to run when IO completes) and immediately returns control to the event loop. The event loop monitors all pending IO operations and runs callbacks as they complete.

```
Request A arrives → parse → start DB query → register callback → return control
(event loop)      → Request B arrives → parse → start DB query → register callback → return control
(event loop)      → checks IO status in a loop using epoll/kqueue
                  → DB response for B ready → run B's callback → return response to B
                  → DB response for A ready → run A's callback → return response to A
```

**OS primitives that make this possible:**
- Linux: `epoll` — efficiently monitors thousands of sockets/file descriptors for IO readiness
- macOS: `kqueue`
- Windows: IOCP

### Callbacks → Promises → async/await

The three are the same mechanism with progressively better syntax:

**Callback style** (old JavaScript):
```javascript
db.query("SELECT * FROM users WHERE id = ?", [userId], function(err, user) {
    // This function runs when the DB responds
    sendResponse(res, user);
});
```

**Problem:** Multiple nested IO calls → deeply nested callbacks ("callback hell").

**async/await style** (modern JavaScript, readable):
```javascript
async function handleRequest(req, res) {
    const user = await db.getUser(userId);   // give up CPU, resume when done
    const orders = await db.getOrders(userId); // give up CPU, resume when done
    return { user, orders };
}
```

async/await is **syntactic sugar** over callbacks — the runtime transforms it into a state machine internally.

### async/await as a State Machine

An `async` function is compiled into a state machine where each `await` creates a new state. Between states, the event loop is free to run other callbacks.

```
State 0: Call db.getUser() → register callback (state 1) → give up control
State 1: [DB responded] → assign to `user` → call db.getOrders() → register callback (state 2) → give up control
State 2: [DB responded] → assign to `orders` → return result
```

This explains two common rules:
- **Why `await` only works inside `async` functions**: the function must be compiled as a state machine
- **Why blocking the event loop is bad**: a blocking CPU operation inside an async function prevents the state machine from ever reaching the next state, freezing all other callbacks

### Event Loop Efficiency

| Property | Detail |
|----------|--------|
| **No context switch overhead** | Single thread — nothing to switch |
| **No stack-per-connection memory** | One thread for all requests |
| **Very efficient for IO-bound work** | Handles thousands of concurrent connections with minimal resources |
| **Poor for CPU-bound work** | Heavy computation blocks the single thread, freezing all other requests |

---

## Mechanism 3: Virtual Threads / Goroutines (Go's Model)

Go takes a third approach — it creates a **scheduler inside the Go runtime** that sits on top of OS threads. This gives the lightweight concurrency of event loops with the familiar blocking code style of threads.

### Architecture

```
OS Level:   [M1 thread] [M2 thread] [M3 thread] [M4 thread]
             ↑ GOMAXPROCS = number of CPU cores (4 by default)

Go Runtime: [G1] [G2] [G3] ... [G1000]  ← goroutines (lightweight)
             Goroutines are distributed across M threads by Go scheduler
```

- **M (Machine):** OS-level threads — one per CPU core by default (`GOMAXPROCS`)
- **G (Goroutine):** Virtual threads — created by Go runtime, not OS. Thousands to millions can exist
- **Go Scheduler:** Assigns goroutines to M threads; handles pausing, switching, and resuming

**Why goroutines are lightweight:**
- Initial stack: ~2–8KB (vs 8MB for OS threads) — grows dynamically
- Switching between goroutines = pointer swap, not full OS context switch
- No system call required to create a goroutine

### How Go Handles IO

When a goroutine blocks on IO:
1. Go runtime detects the block
2. Parks (pauses) that goroutine
3. The OS thread (M) picks up another goroutine from the queue
4. When IO completes, the parked goroutine becomes runnable again

This is why Go's HTTP server creates a **new goroutine per request** — goroutines are cheap enough to justify it:

```go
// From Go's standard library (simplified)
func (srv *Server) Serve(l net.Listener) {
    for {
        conn, _ := l.Accept()
        go srv.ServeHTTP(conn, handler)  // new goroutine per request
    }
}
```

### Go Code — No Callbacks Required

```go
func handleRequest(w http.ResponseWriter, r *http.Request) {
    user, err := db.Query("SELECT * FROM users WHERE id = $1", userID)
    // goroutine pauses here automatically, Go scheduler runs other goroutines
    // resumes when DB responds — no callback syntax needed
    if err != nil { /* handle */ }
    json.NewEncoder(w).Encode(user)
}
```

Code looks synchronous but behaves concurrently at the goroutine level.

---

## When to Use Each

| Workload | Best approach | Why |
|---------|--------------|-----|
| **IO-bound** (web servers, APIs, DB-heavy) | Concurrency (async/await, goroutines) | CPU mostly idle — avoid thread-per-request overhead |
| **CPU-bound** (image processing, encryption, ML) | Parallelism (multiple OS threads, worker pools) | Want true simultaneous execution across cores |
| **Mixed** (most real backends) | Both — goroutines for IO, worker pools for CPU-heavy tasks | Each tool for the right job |

---

## Concurrency Problems: Race Conditions

Concurrency introduces shared state bugs. The primary source is multiple concurrent tasks accessing and modifying the same variable.

### The Lost Update Problem (Threads)

Two threads increment a shared counter from 0:

```
ms 1: Thread A reads counter = 0
ms 2: Thread B reads counter = 0  (reads before A writes)
ms 3: Thread A adds 1 → register = 1
ms 4: Thread B adds 1 → register = 1
ms 5: Thread A writes counter = 1
ms 6: Thread B writes counter = 1  ← overwrites A's result!

Expected: 2    Actual: 1
```

One update is lost. This is a **race condition** — the outcome depends on the exact timing of operations.

### Race Conditions in Event Loops (async/await)

Single-threaded event loops are not immune. Example — a withdrawal function:

```javascript
let balance = 100;

async function withdraw(amount) {
    if (balance >= amount) {        // ← check happens here
        await processWithdrawal();  // ← gives up control here
        balance -= amount;          // ← update happens here
    }
}

withdraw(100);  // call 1
withdraw(100);  // call 2
```

**The bug:**
1. Call 1 checks: `100 >= 100` → true, enters block
2. Call 1 hits `await` → gives up control to event loop
3. Call 2 checks: `100 >= 100` → true (balance not yet updated!), enters block
4. Call 2 hits `await` → gives up control
5. Call 1 resumes: `balance = 100 - 100 = 0`
6. Call 2 resumes: `balance = 0 - 100 = -100` ← invalid state

Race conditions can occur anywhere the event loop switches between tasks (at every `await`).

### Solutions to Race Conditions

| Solution | How it works | Language/context |
|----------|-------------|-----------------|
| **Locks / Mutexes** | Only one thread enters a critical section at a time; others wait | Python `threading.Lock`, Java `synchronized` |
| **Channels** | Goroutines communicate by message-passing instead of shared memory; one goroutine owns the variable | Go |
| **Atomic operations** | Hardware-level instructions that read-modify-write in one uninterruptible step | Most languages |
| **Redesign** | Move state into a database where atomic transactions handle consistency | Universal |

> The deeper you go into concurrency, the more important it becomes to minimize shared mutable state. Prefer designs where each concurrent unit owns its data exclusively.

---

## Summary

| Concept | Key point |
|---------|----------|
| **IO-bound** | Waiting for external resources (DB, network, files) — CPU is idle |
| **CPU-bound** | Active computation (parsing, encryption, image processing) — CPU is busy |
| **Concurrency** | Dealing with multiple things at once; can use 1 core; about structure |
| **Parallelism** | Doing multiple things at the same instant; requires multiple cores; about hardware |
| **Threads** | OS-managed; heavy (memory, creation, context switch); good for parallelism |
| **Event loop** | Single thread + callbacks; lightweight; excellent for IO-bound; blocks on CPU work |
| **Goroutines** | Go's virtual threads; lightweight (2KB stack, no syscall); bridges both worlds |
| **async/await** | Syntactic sugar over callbacks; compiles to state machine; same event loop underneath |
| **Race conditions** | Shared mutable state + concurrent access = unpredictable bugs; affects threads AND async/await |
| **Most backends** | IO-bound → concurrency primitives are the right tool |
