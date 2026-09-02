# Table-of-Contents

<!-- toc -->

- [Graceful Shutdown](#graceful-shutdown)
  * [The Problem](#the-problem)
  * [Process Lifecycle Management](#process-lifecycle-management)
  * [OS Signals](#os-signals)
    + [SIGTERM — Polite Termination Request](#sigterm--polite-termination-request)
    + [SIGINT — Interrupt](#sigint--interrupt)
    + [SIGKILL — Force Kill](#sigkill--force-kill)
  * [Two Steps of Graceful Shutdown](#two-steps-of-graceful-shutdown)
  * [Step 1 — Connection Draining](#step-1--connection-draining)
    + [The Restaurant Analogy](#the-restaurant-analogy)
    + [Architecture-Specific Implementations](#architecture-specific-implementations)
    + [The Timeout](#the-timeout)
  * [Step 2 — Resource Cleanup](#step-2--resource-cleanup)
    + [Common Resources to Clean Up](#common-resources-to-clean-up)
    + [Cleanup Order — Reverse Acquisition Order](#cleanup-order--reverse-acquisition-order)
  * [Graceful Shutdown Flow (Go Example)](#graceful-shutdown-flow-go-example)
  * [Process Lifecycle Summary](#process-lifecycle-summary)
  * [Why It Matters](#why-it-matters)

<!-- tocstop -->

---

# Graceful Shutdown

**Source:** Sriniously — Backend from First Principles (Video 19)
**Link:** [Watch](https://www.youtube.com/watch?v=estH64OkwxU)

---

## The Problem

During a deployment, a server must restart. Even with zero-downtime deployment strategies (where the new server comes up before the old one goes down), there is a critical moment when the old server must stop receiving traffic and hand off to the new one.

If the old server is mid-transaction at that moment (e.g. processing a payment), what happens?
- Does the transaction get lost?
- Does the customer get charged twice due to a race condition?
- Is data corrupted?

**Graceful shutdown** solves this. The goal: teach your server **good manners** — finish what it's doing, clean up, then exit. Don't slam the door.

---

## Process Lifecycle Management

Every backend application runs as a **process** inside an operating system. Like all living things, a process has a lifecycle:
- **Born** — process starts
- **Lives** — process executes
- **Dies** — process terminates

How a process is terminated matters enormously. The OS doesn't just pull the plug — it follows an **established communication protocol** using **signals**.

---

## OS Signals

Signals are the mechanism Unix/Linux operating systems use for **inter-process communication (IPC)** — how the OS communicates with a running process.

Your application registers **signal handlers** — code that waits in the background and responds when specific signals arrive.

### SIGTERM — Polite Termination Request

- A gentle nudge: "Hey, could you please finish up and leave?"
- Gives your application an opportunity to complete in-flight work and clean up
- **Used by:** deployment systems, process managers (PM2, systemd), orchestration platforms (Kubernetes)
- **Can be caught** — your application can register a handler and respond gracefully

### SIGINT — Interrupt

- Triggered by **Ctrl + C** in the terminal
- A developer-initiated shutdown signal
- **Can be caught** — handle the same way as SIGTERM
- **Used by:** developers during local development

> Both SIGTERM and SIGINT should be handled identically — the intention is the same (shut down cleanly), only the initiator differs (program vs human).

### SIGKILL — Force Kill

- **Cannot be caught, cannot be ignored** — the process is immediately terminated
- No opportunity for cleanup — equivalent to pulling the power plug
- **Used as a last resort** when the process doesn't respond to SIGTERM within the timeout

> If you don't respect SIGTERM/SIGINT (the polite signals), you will eventually receive SIGKILL. At that point, you have no chance to clean up.

---

## Two Steps of Graceful Shutdown

When SIGTERM or SIGINT is received, a gracefully shutting down server performs two main steps:

1. **Connection draining** — finish in-flight requests
2. **Resource cleanup** — release all acquired resources

Then **exit**.

---

## Step 1 — Connection Draining

Your HTTP server processes multiple requests concurrently. When it receives a shutdown signal, some requests are already mid-execution — these are called **in-flight** or **on-the-fly requests**.

### The Restaurant Analogy

When a restaurant closes:
1. **Stop letting new customers in** — no new orders
2. **Give existing customers time to finish** — announce 15–20 minutes to wrap up
3. **Close the restaurant** — everyone leaves, lights off

Connection draining follows the same three steps:

```
SIGTERM received
    │
    ├─ 1. Stop accepting new connections/requests
    │       (no new customers)
    │
    ├─ 2. Wait for in-flight requests to complete
    │       (existing customers finish their meal)
    │
    └─ 3. Close connections and exit
```

### Architecture-Specific Implementations

| Server type | What connection draining means |
|-------------|-------------------------------|
| HTTP server | Stop accepting new HTTP requests; let in-flight requests complete |
| Database | Commit or rollback existing transactions; stop accepting new queries |
| WebSocket | Notify connected clients the server is closing; then close sockets |

### The Timeout

You cannot wait indefinitely for in-flight requests. Most production systems set a **hard timeout** (commonly 30 seconds):

- After SIGTERM: stop new connections, wait up to 30s for existing ones to finish
- If any requests are still running after the timeout → forcefully terminate

**Choosing the timeout:**
- Too short → legitimate operations get interrupted
- Too long → deployments become sluggish, system responsiveness suffers
- Base it on your application's typical request duration and operational needs
- 30s is a reasonable default for standard HTTP APIs; adjust for long-running operations (WebSockets, AI generation, etc.)

> Connection draining also requires coordination with your **load balancer** and **service discovery** system — they need to stop routing new traffic to the shutting-down instance.

---

## Step 2 — Resource Cleanup

When your backend was running, it acquired various system resources. These must be explicitly released on shutdown.

### Common Resources to Clean Up

| Resource | Why it matters |
|----------|---------------|
| **File handles** | Each open file consumes memory/OS handles; leaking them exhausts the process's handle limit |
| **Database connections** | Uncommitted transactions left open can cause deadlocks or data corruption |
| **Network connections** | Open TCP connections consume OS resources; must be explicitly closed |
| **Redis connections** | Background job queues and caches hold connections that need graceful closure |
| **Temporary files** | Files created during execution should be deleted on exit |

### Cleanup Order — Reverse Acquisition Order

**Always clean up resources in the reverse order you acquired them.**

Example startup order:
1. Redis connection
2. Database connection pool
3. HTTP server

Shutdown cleanup order:
1. HTTP server (stop accepting traffic first)
2. Database connection pool (finish transactions, close connections)
3. Redis connection (close background job workers)

This prevents cleanup of a resource that another still-active resource depends on.

---

## Graceful Shutdown Flow (Go Example)

```
Register signal handlers for SIGTERM and SIGINT
    │
    ▼
Signal received (Ctrl+C or deployment manager)
    │
    ▼
Call graceful shutdown function:
    1. Shutdown HTTP server
       → library stops accepting new requests
       → waits for in-flight requests to complete
    2. Close database connections
       → finish existing transactions
       → close all TCP connections in the pool
    3. Shutdown background job server (Asynq/BullMQ/etc.)
       → waits for all workers to finish current tasks
       → closes Redis connections
    │
    ▼
Log "server exited properly"
```

**Startup logs:**
```
Connected to database
Starting background job server
Server started on :8080
```

**Shutdown logs (after Ctrl+C):**
```
Interrupt signal received
Closing database connection
Starting gracefully shutdown (background jobs)
Waiting for all workers to finish
All workers have finished
Server has exited properly
```

---

## Process Lifecycle Summary

```
Start
  ↓ register signal handlers
  ↓ validate config
  ↓ connect to resources (Redis, DB, etc.)
  ↓ start HTTP server

Running
  ↓ process requests
  ↓ execute background jobs

SIGTERM / SIGINT received
  ↓ stop accepting new connections
  ↓ drain in-flight requests (up to timeout)
  ↓ cleanup resources (reverse acquisition order)
  ↓ exit cleanly

(If SIGKILL received instead → immediate forced termination, no cleanup)
```

---

## Why It Matters

| Without graceful shutdown | With graceful shutdown |
|--------------------------|----------------------|
| In-flight payments may be lost or double-charged | Transactions complete before shutdown |
| Database transactions left in inconsistent state | Transactions committed or rolled back cleanly |
| File handles and connections leak | All resources released properly |
| Deadlocks on next startup | Clean state for new instance |
| Poor user experience during deployments | Seamless to end users |

Most frameworks provide built-in graceful shutdown support — use it. The key is understanding **why** it's needed and **what** it does, so you configure it correctly for your application's specific workloads and timeouts.
