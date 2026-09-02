# Table-of-Contents

<!-- toc -->

- [Logging, Monitoring and Observability](#logging-monitoring-and-observability)
  * [The Mindset](#the-mindset)
  * [The Three Terms](#the-three-terms)
    + [Logging](#logging)
    + [Monitoring](#monitoring)
    + [Observability](#observability)
  * [How the Three Pillars Work Together](#how-the-three-pillars-work-together)
  * [Logging in Depth](#logging-in-depth)
    + [Log Levels](#log-levels)
    + [Structured vs Unstructured Logging](#structured-vs-unstructured-logging)
    + [What to Log](#what-to-log)
  * [Metrics](#metrics)
  * [Traces](#traces)
    + [Instrumentation and OpenTelemetry](#instrumentation-and-opentelemetry)
  * [Implementation Pattern (Go example)](#implementation-pattern-go-example)
    + [Middleware — Create Transaction](#middleware--create-transaction)
    + [Service Layer — Use Transaction](#service-layer--use-transaction)
  * [Tools](#tools)
    + [Open Source (Self-Hosted)](#open-source-self-hosted)
    + [Proprietary / Managed](#proprietary--managed)
  * [Summary](#summary)

<!-- tocstop -->

---

# Logging, Monitoring and Observability

**Source:** Sriniously — Backend from First Principles (Video 18)
**Link:** [Watch](https://www.youtube.com/watch?v=estH64OkwxU)

---

## The Mindset

Logging, monitoring and observability are **practices**, not a binary on/off state. No system can be said to be 100% observable — it exists on a spectrum. The goal is to continuously improve coverage.

These practices matter because modern backends run in distributed environments across multiple servers, regions, and services. Without them, you know your system is broken only after users complain. With them, you detect and diagnose issues before they cause major damage.

---

## The Three Terms

### Logging

**Recording** all important events across the full lifecycle of your application — startup, request handling, errors, database queries, business operations.

> Like a journal or diary your backend maintains. When something goes wrong, you know what happened, when, and why.

### Monitoring

**Continuously tracking** the health and performance of your system in near-real-time (typically 10–15 second delay).

Tracks things like:
- Requests processed per second
- Error rates
- CPU / memory usage
- Open database connections

Monitoring tells you **that** there is a problem.

### Observability

A system is **observable** if you can determine its internal state by examining its external outputs. Observability tells you **what** is wrong and **why**.

Observability has three pillars:

| Pillar | What it gives you |
|--------|------------------|
| **Logs** | What happened — full event record with context |
| **Metrics** | Patterns and trends — quantified system state over time |
| **Traces** | Component interactions — the full path of a request through your system |

> **Monitoring (traditional):** "Something is wrong."
> **Observability:** "Something is wrong, here's exactly where and why."

---

## How the Three Pillars Work Together

A typical debugging workflow:

```
Alert fires (error rate > 80%)
    │
    ▼
Metrics dashboard
    (see: error count spiked at 3:15pm, GET /todos endpoint)
    │
    ▼
Related logs
    (see: "401 Unauthorized" errors from specific IP range)
    │
    ▼
Trace for a specific failed request
    (see: request entered middleware → validation passed → service method failed at line X)
    │
    ▼
Root cause identified → fix deployed
```

---

## Logging in Depth

### Log Levels

| Level | When to use | Environment |
|-------|------------|-------------|
| `debug` | Detailed troubleshooting, verbose output | Development only |
| `info` | General operations, successful business events (user created, to-do saved) | Dev + prod |
| `warn` | Non-critical issues (auth failed, retrying operation) — not our fault, not an error | Dev + prod |
| `error` | Actual failures — validation errors, DB query failures, unhandled exceptions | Dev + prod |
| `fatal` | Critical failure — application is shutting down | Dev + prod |

> Set log level to `debug` in development, `info` in production. This filters out noisy debug logs from your valuable production log stream.

### Structured vs Unstructured Logging

| Type | Format | Use in |
|------|--------|--------|
| **Unstructured** | Human-readable plain text, colored output | Development (local) |
| **Structured** | JSON with explicit fields (`level`, `message`, `userId`, `requestId`, etc.) | Production |

**Why JSON in production:**
- Log management tools (ELK stack, Loki, Grafana, New Relic) parse and index JSON efficiently
- Plain text is hard to machine-parse — extracting user IDs, request IDs, timestamps becomes unreliable
- Enables searching, filtering, and alerting on specific fields

**Why plain text in development:**
- JSON logs are hard to read at a glance
- Colored, formatted output makes it easy to spot errors quickly

### What to Log

At minimum, include these fields in every log entry:
- `timestamp`
- `level`
- `message`
- `userId` (never email or password)
- `requestId` (correlation ID for tracing across services)
- `method` and `route` (for request logs)
- Error details (for error logs — type, message, but not raw DB internals)

---

## Metrics

Metrics are **quantified, trackable numbers** about your system — both current state and historical trends.

**Examples:**
- Total requests processed
- Request success / failure rate
- Average response time per endpoint
- Number of DB connections in use
- Business metrics: to-dos created, orders placed, payments processed

Metrics are what you display on dashboards and configure alerts against.

---

## Traces

A **trace** represents the full lifecycle of a single request as a transaction — tracking every component it touches from entry to exit.

**Example trace for `POST /todos`:**
```
Request received (middleware layer) → t=0ms
  Auth middleware → t=2ms
  Validation → t=3ms
  Service: createTodo() → t=5ms
    Repository: insertTodo() → t=6ms
    DB query executed → t=8ms
  Response sent → t=10ms
```

Each segment of a trace is called a **span**. Together they form the full picture of what your system did for that request.

### Instrumentation and OpenTelemetry

**Instrumentation** = the practice of measuring different attributes of your code as it runs.

**OpenTelemetry** = an open standard with SDKs for all major languages that defines how to instrument applications for traces, metrics, and logs. Works with both open-source and proprietary backends.

---

## Implementation Pattern (Go example)

### Middleware — Create Transaction

The first middleware in the chain creates a new trace transaction and attaches metadata to it, then stores it in the request context:

```go
// Enhanced tracing middleware
transaction := newrelic.StartTransaction(serviceName)
transaction.AddAttribute("userId", userId)
transaction.AddAttribute("requestId", requestId)
transaction.AddAttribute("environment", env)
ctx = context.WithValue(ctx, "transaction", transaction)
next.ServeHTTP(w, r.WithContext(ctx))
```

### Service Layer — Use Transaction

Each service method retrieves the transaction from context, adds its own attributes, and ends its span when done:

```go
func (s *Service) CreateTodo(ctx context.Context, ...) {
    txn := ctx.Value("transaction").(*newrelic.Transaction)
    defer txn.End() // end this span when function returns

    txn.AddAttribute("userId", userId)
    txn.AddAttribute("todoTitle", title)

    s.logger.Info("creating todo", "title", title)

    // on success
    txn.AddAttribute("operation", "create")
    txn.AddAttribute("todoId", newId)
    s.logger.Debug("todo created successfully", "id", newId)

    // business event log
    s.logger.Info("todo.created", "id", newId, "title", title, "priority", priority)

    // on error
    s.logger.Error("failed to create todo", "error", err)
    txn.NoticeError(err)
}
```

---

## Tools

### Open Source (Self-Hosted)

| Tool | Role |
|------|------|
| **Prometheus** | Metrics collection and storage |
| **Grafana** | Dashboards and alerting |
| **Loki** | Log aggregation |
| **Promtail** | Log shipping agent (sends logs to Loki) |
| **Jaeger** | Distributed tracing |

The **Grafana stack** (Prometheus + Grafana + Loki + Promtail + Jaeger) is the most common open-source observability setup.

### Proprietary / Managed

| Tool | Notes |
|------|-------|
| **New Relic** | All-in-one — logs, metrics, traces in one platform |
| **Datadog** | Full observability platform |

**When to use proprietary:** When you don't have the team or resources to configure and maintain multiple open-source tools. Proprietary solutions simplify setup at the cost of vendor lock-in and pricing.

---

## Summary

| Concept | Key point |
|---------|----------|
| **Logging** | Record all important events with context — structured JSON in production, readable text in dev |
| **Log levels** | debug → info → warn → error → fatal; filter by environment |
| **Monitoring** | Real-time metrics — error rates, throughput, resource usage |
| **Traces** | Full request lifecycle across all components — essential for debugging distributed systems |
| **Observability** | The combination of all three — tells you what failed, where, and why |
| **Instrumentation** | Measuring code attributes at runtime; OpenTelemetry is the standard |
| **It's a spectrum** | No system is 100% observable — continuously improve coverage |
| **Team effort** | Developers implement logging/tracing in code; DevOps sets up collection and dashboards |
