# Table-of-Contents

<!-- toc -->

- [Background Jobs and Task Queues](#background-jobs-and-task-queues)
  * [What is a Background Task?](#what-is-a-background-task)
  * [Why Background Tasks Exist — The Email Example](#why-background-tasks-exist--the-email-example)
    + [Synchronous (no background task)](#synchronous-no-background-task)
    + [Asynchronous (with background task)](#asynchronous-with-background-task)
  * [How Task Queues Work](#how-task-queues-work)
    + [The Three Components](#the-three-components)
    + [Acknowledgements](#acknowledgements)
    + [Visibility Timeout](#visibility-timeout)
    + [Retry with Exponential Backoff](#retry-with-exponential-backoff)
  * [Technologies](#technologies)
    + [Queue / Broker Technologies](#queue--broker-technologies)
    + [Task Processing Frameworks](#task-processing-frameworks)
  * [Types of Background Tasks](#types-of-background-tasks)
    + [1. One-Off Tasks](#1-one-off-tasks)
    + [2. Recurring Tasks](#2-recurring-tasks)
    + [3. Chain Tasks (Parent-Child)](#3-chain-tasks-parent-child)
    + [4. Batch Tasks](#4-batch-tasks)
  * [Common Use Cases in SaaS Backends](#common-use-cases-in-saas-backends)
  * [Design Considerations at Scale](#design-considerations-at-scale)
    + [Idempotency](#idempotency)
    + [Error Handling](#error-handling)
    + [Monitoring](#monitoring)
    + [Horizontal Scaling](#horizontal-scaling)
    + [Ordered Delivery](#ordered-delivery)
    + [Rate Limiting](#rate-limiting)
  * [Best Practices](#best-practices)
  * [Summary](#summary)

<!-- tocstop -->

---

# Background Jobs and Task Queues

**Source:** Sriniously — Backend from First Principles (Video 14)
**Link:** [Watch](https://www.youtube.com/watch?v=estH64OkwxU)

---

## What is a Background Task?

A **background task** = any piece of code that runs **outside of the request-response lifecycle**.

```
Client ──── request ───→ Server ──── response ───→ Client
              (everything outside this = background task)
```

Key characteristics:
- Does **not** need to happen immediately — not mission-critical in real time
- Is **not synchronous** — can be offloaded to a separate process
- Executes however it has been programmed to finish, independently of the original request

---

## Why Background Tasks Exist — The Email Example

### Synchronous (no background task)

User signs up → server validates → server calls email provider API → (if API fails) → signup API itself fails or shows stale state → bad user experience.

**Problems:**
- If the email provider is down, the signup API fails or lies ("email sent" when it wasn't)
- User is blocked; must manually retry
- Latency of external API call bleeds into your API response time

### Asynchronous (with background task)

```
User signs up
  → server validates
  → server serializes email data into JSON
  → pushes task into queue  ← only this happens in the request lifecycle
  → immediately returns 200/201 to client
  → client sees "verification email sent" instantly

Meanwhile (in a separate process):
  → consumer picks task from queue
  → deserializes data
  → calls email provider API
  → if fails → retries with exponential backoff
  → email delivered successfully
```

**Benefits:**
- API response is instant and always succeeds
- Email delivery failures are handled silently with automatic retries
- User experience is unaffected by third-party downtime

---

## How Task Queues Work

```
Producer (your app code)
    │
    │  serialize task to JSON
    │  enqueue (ENQ)
    ▼
[  Queue / Broker  ]   ← temporary holding area (RabbitMQ, Redis, SQS)
    │
    │  dequeue (DEQ)
    ▼
Consumer / Worker (separate process)
    │
    │  deserialize JSON → native format
    │  execute registered handler
    │  send acknowledgement (ACK) back to queue
    ▼
 Task complete → removed from queue
```

### The Three Components

| Component | Role |
|-----------|------|
| **Producer** | Your app code — creates task, serializes to JSON, pushes to queue (ENQ) |
| **Broker / Queue** | Stores tasks until a worker is ready — RabbitMQ, Redis PubSub, AWS SQS |
| **Consumer / Worker** | Separate process — monitors queue, DEQs tasks, executes handlers, sends ACK |

> Think of it as a **to-do list for your backend** — the app adds tasks, workers pick them off one by one.

### Acknowledgements

- After completing a task, the consumer sends an **ACK** (acknowledgement) to the queue
- ACK = task successfully processed → queue removes it
- No ACK received → queue assumes failure → initiates retry

### Visibility Timeout

The period during which a task is considered **in progress** by a consumer.

- If no ACK arrives within the timeout (consumer crashed, external service hung), the queue makes the task **available to other consumers**
- Prevents tasks from being lost if a worker dies mid-execution

### Retry with Exponential Backoff

When a task fails, it is re-injected into the queue with increasing delays:

| Attempt | Wait before retry |
|---------|-------------------|
| 1st fail | 1 minute |
| 2nd fail | 2 minutes |
| 3rd fail | 4 minutes |
| 4th fail | 8 minutes |
| Max retries reached | Dead-letter queue / alert |

External services are rarely down for more than a few minutes — most tasks succeed within 1–2 retries.

---

## Technologies

### Queue / Broker Technologies

| Technology | Notes |
|-----------|-------|
| **RabbitMQ** | Dedicated message broker |
| **Redis PubSub** | Redis publisher/subscriber module — common for task queues |
| **AWS SQS** | Managed queuing service, multi-region, highly scalable |

### Task Processing Frameworks

| Language | Framework |
|----------|-----------|
| Python | **Celery** |
| Node.js | **BullMQ** |
| Go | **Asynq** |

All provide: task creation, queue management, retries, exponential backoff, scheduled tasks, failure detection.

---

## Types of Background Tasks

### 1. One-Off Tasks

Triggered by a specific event — execute once.

**Examples:**
- Send verification email on signup
- Send welcome email after verification
- Send password reset email
- Send push notification when a user receives a message

### 2. Recurring Tasks

Execute periodically on a schedule (cron jobs).

**Examples:**
- Send daily/weekly/monthly reports to users
- Database cleanup — delete orphan sessions from the sessions table every month
- Cache warm-up, index rebuilding, metrics aggregation

> Most frameworks (Celery, BullMQ, Asynq) have built-in **scheduled task** support for periodic execution.

### 3. Chain Tasks (Parent-Child)

Tasks with dependencies — a task can only start after its parent succeeds.

**Example — LMS video upload workflow:**

```
Video uploaded to S3
    │
    ▼
[Task 1] Encode video to multiple resolutions
    │
    ├──→ [Task 2] Generate thumbnails
    │         │
    │         ▼
    │    [Task 3] Process thumbnail images (multiple resolutions)
    │
    └──→ [Task 4] Generate audio transcription (subtitles)
         (runs in parallel with Task 2 — no dependency between them)
```

Task 2 and Task 4 both depend on Task 1 but are independent of each other → run in parallel.

### 4. Batch Tasks

One task triggers many tasks, or many tasks of the same type run simultaneously.

**Examples:**
- **Delete account** — triggers sub-tasks: delete projects, remove assets, delete profile, send confirmation email
- **Send weekly reports** — triggers thousands of report-generation tasks simultaneously for all users

---

## Common Use Cases in SaaS Backends

| Use Case | Why background? |
|----------|----------------|
| **Sending emails** | Depends on external service (Resend, Mailgun, Brevo) |
| **Image / video processing** | CPU-intensive — resizing, encoding, format conversion |
| **Report generation** | Heavy DB queries + PDF/HTML construction |
| **Push notifications** | Depends on external OS service (Google FCM, Apple APNs) |
| **Account deletion** | Too many DB operations to fit in one request |
| **Scheduled reports** | Must run at a specific time, not on-demand |

---

## Design Considerations at Scale

### Idempotency

Design tasks so they can be **safely re-executed multiple times** without causing side effects.

- Wrap delete/update operations in a **database transaction**
- On failure → rollback to clean state
- On retry → start from scratch without corrupting data

### Error Handling

- Implement **robust and extensive error handling** — tasks run in a separate process with no human watching
- Log all errors with enough context to debug (task ID, input data, failure reason)
- Handle both internal errors and external service failures explicitly

### Monitoring

Track at all times:
- Number of tasks in queue (queue depth)
- Number of successful vs failed tasks
- Root cause of failures (external service? internal bug?)

**Tools:** Prometheus + Grafana, ELK Stack (covered in a future video on observability)

### Horizontal Scaling

Design consumers so you can **add more worker nodes** as load increases — stateless workers scale horizontally without changes to the queue.

### Ordered Delivery

If task order matters, verify your library/broker supports **ordered delivery** guarantees.

### Rate Limiting

If tasks call external APIs, implement rate limiting to avoid overloading those services and violating their quotas.

---

## Best Practices

| Practice | Why |
|----------|-----|
| **Keep tasks small and focused** | One task = one unit of work; easier to retry, scale, monitor, and debug |
| **Avoid long-running tasks** | Break into smaller chained or parallel tasks |
| **Robust error handling + logging** | Tasks run silently in background — you need observability |
| **Monitor queue length and worker health** | Set alerts when queue depth spikes or workers go down |
| **Design for idempotency** | Tasks will be retried — they must be safe to run multiple times |

---

## Summary

| Concept | Key point |
|---------|-----------|
| **Background task** | Runs outside request-response lifecycle |
| **Producer** | Your app — creates and ENQueues tasks |
| **Broker** | Queue storage — RabbitMQ, Redis, SQS |
| **Consumer/Worker** | Separate process — DEQueues and executes tasks |
| **ACK** | Signal from worker to queue: task done, remove it |
| **Visibility timeout** | Grace period before queue reassigns an unacknowledged task |
| **Exponential backoff** | Retry delays double on each failure |
| **One-off** | Triggered by event, runs once |
| **Recurring** | Scheduled on cron interval |
| **Chain** | Parent-child dependency between tasks |
| **Batch** | One trigger spawns many tasks |
