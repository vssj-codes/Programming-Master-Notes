# Table-of-Contents

<!-- toc -->

- [Controllers, Services, Repositories, Middlewares and Request Context](#controllers-services-repositories-middlewares-and-request-context)
  * [The Request Life Cycle Inside the Server](#the-request-life-cycle-inside-the-server)
  * [Handlers / Controllers, Services, and Repositories](#handlers--controllers-services-and-repositories)
    + [Handler / Controller Layer](#handler--controller-layer)
    + [Service Layer](#service-layer)
    + [Repository / Database Layer](#repository--database-layer)
    + [The Full Flow](#the-full-flow)
  * [Middlewares](#middlewares)
    + [What a Middleware Receives](#what-a-middleware-receives)
    + [Why Middlewares Exist](#why-middlewares-exist)
    + [How They Work](#how-they-work)
    + [Common Middleware Examples](#common-middleware-examples)
      - [Security Middlewares](#security-middlewares)
      - [Other Common Middlewares](#other-common-middlewares)
    + [Middleware Ordering (Typical)](#middleware-ordering-typical)
  * [Request Context](#request-context)
    + [What It Is](#what-it-is)
    + [Why It Exists](#why-it-exists)
    + [Common Use Cases](#common-use-cases)
      - [1. Authentication Data](#1-authentication-data)
      - [2. Request ID for Tracing](#2-request-id-for-tracing)
      - [3. Cancellation Signals and Deadlines](#3-cancellation-signals-and-deadlines)
  * [Summary](#summary)
    + [Key Principles](#key-principles)

<!-- tocstop -->

---

# Controllers, Services, Repositories, Middlewares and Request Context

**Source:** Sriniously — Backend from First Principles (Video 10)
**Link:** [Watch](https://www.youtube.com/watch?v=hyc-7w3pee8)

---

## The Request Life Cycle Inside the Server

```
Client → Entry Point → Middlewares → Routing → Middlewares → Handler/Controller → Service → Repository → DB
                                                                    ↓
Client ← Response ← Error Middleware ← ──────────────────── ← Controller ←
```

When the OS forwards an HTTP request to the port your server is listening on, that's the **entry point**. From there, the request flows through a chain of processing layers until a response is sent back.

---

## Handlers / Controllers, Services, and Repositories

These three layers are a **design pattern** — not a hard requirement. You *can* put everything in one function, but separating responsibilities makes the codebase scalable, maintainable, and debuggable.

### Handler / Controller Layer

**Receives:** `request` and `response` objects (provided by the runtime/framework).

**Responsibilities — all HTTP-related concerns:**

1. **Binding / Deserialization** — extract data from the request object (body, query params, path params) and deserialize JSON into native data types (Go struct, Python dict, Rust struct). If this fails → `400 Bad Request`.

2. **Validation & Transformation** — verify the data matches expected format, types, constraints. Set defaults (e.g. `sort = "date"` if not provided). If validation fails → `400 Bad Request`.

3. **Call service layer** — pass the validated, transformed data along with auth metadata (user ID, role) to the service method.

4. **Send response** — based on service result, choose the appropriate status code (`200`, `201`, `204`, `400`, `500`) and send the response body back to the client.

> The controller **controls the data flow** — from client to server and back. All HTTP logic lives here.

### Service Layer

**Receives:** validated data + metadata (user ID, permissions, etc.)

**Responsibilities — all business logic:**

- Orchestrates operations: calls repository methods, sends emails, fires notifications, makes external API calls, webhooks
- Can call **multiple repository methods** and merge/transform their results
- Returns data to the controller

> **Key principle:** A service method should look like a plain function. Looking at it, you should **not be able to tell** it's being used in an API. No HTTP status codes, no request/response objects — just data in, processing, data out.

### Repository / Database Layer

**Receives:** data needed for the database operation (filters, sort params, values to insert)

**Responsibilities — database operations only:**

- Constructs and executes database queries
- Returns the raw result from the database

> **Single responsibility:** One repository method = one type of operation. Don't make a method that conditionally returns all books OR a single book based on an optional parameter. Split them: `getAllBooks()` and `getBookById(id)`.

### The Full Flow

```
Controller                          Service                        Repository
    │                                  │                               │
    │ 1. Bind/deserialize request      │                               │
    │ 2. Validate & transform          │                               │
    │ 3. Call service ─────────────→   │                               │
    │                                  │ Execute business logic        │
    │                                  │ Call repository ──────────→   │
    │                                  │                               │ Execute DB query
    │                                  │                               │ Return result
    │                                  │ ←────────────────────────     │
    │                                  │ Orchestrate, merge data       │
    │                                  │ Send email, notifications     │
    │ ←────────────────────────────    │ Return processed data         │
    │ 4. Choose status code            │                               │
    │ 5. Send response to client       │                               │
```

---

## Middlewares

Functions that execute **in between** processing boundaries — between the entry point and the handler, and between the handler and the response.

### What a Middleware Receives

Every middleware gets three things from the runtime:

| Parameter | Purpose |
|-----------|---------|
| `request` | Read/modify incoming request data |
| `response` | Send response back or modify response headers |
| `next` | Pass execution to the next middleware or handler |

### Why Middlewares Exist

Without middlewares, you'd duplicate common operations (auth checks, logging, CORS headers) in **every single handler** across hundreds/thousands of endpoints. Middlewares eliminate this duplication — write the logic once, execute it for every request.

### How They Work

```
Request → MW1 → MW2 → Routing → MW3 → MW4 → Handler → ... → Error MW → Response
              next()  next()           next()  next()                next()
```

- Each middleware does its work, then calls `next()` to pass execution forward
- A middleware can **terminate the request early** by sending a response (e.g. `401 Unauthorized`) without calling `next()`
- Middlewares are **optional** — you add them based on your requirements
- **Order matters** — middlewares execute in the order they're registered

### Common Middleware Examples

#### Security Middlewares

| Middleware | What it does | On failure |
|-----------|-------------|-----------|
| **CORS** | Checks request `Origin` against allowed origins, adds `Access-Control-Allow-Origin` headers | No headers added → browser blocks response |
| **Security Headers** | Adds `Content-Security-Policy`, `X-Frame-Options`, etc. to every response | — |
| **Authentication** | Extracts token (JWT/session ID), verifies credentials, stores user ID + role in request context | `401 Unauthorized` |
| **Rate Limiting** | Checks if client IP exceeded request threshold (e.g. 30 requests/2 seconds) | `429 Too Many Requests` |

#### Other Common Middlewares

| Middleware | What it does |
|-----------|-------------|
| **Logging** | Logs request method, path, query params, body for debugging and auditing |
| **Global Error Handling** | Catches unstructured errors from anywhere in the app, sends properly formatted error response (`400`/`500`) |
| **Compression** | Compresses large responses (gzip) before sending — browsers decompress transparently |
| **Data Parsing** | Handles serialization/deserialization (e.g. `express.json()` body parser in Node.js) |
| **Validation/Transformation** | Can be delegated to middleware instead of doing it in the handler |

### Middleware Ordering (Typical)

```
1. CORS              ← terminate unknown origins early, don't waste resources
2. Security Headers  ← add headers to every response
3. Logging           ← log every request for debugging/auditing
4. Data Parsing      ← deserialize request body
5. Authentication    ← verify credentials, attach user info to context
6. Rate Limiting     ← throttle excessive requests
7. Routing → Handler → Service → Repository
8. Global Error Handling ← LAST — catches errors from ALL upstream layers
```

> **Global error handling goes last** because errors can occur at any point upstream. If you place it in the middle, errors from downstream handlers won't be caught.

---

## Request Context

A **shared state/storage scoped to a single request** — accessible by all middlewares and handlers in that request's lifecycle.

### What It Is

- Typically a key-value store attached to the request
- Each HTTP request gets its own isolated context
- All middlewares and handlers can **read from** and **write to** it
- Prevents tight coupling — middlewares don't need to pass data directly to each other

### Why It Exists

Without context, you'd need to explicitly pass data (user ID, roles, request ID) through every function boundary. Context provides a shared, decoupled storage that any middleware or handler can access.

### Common Use Cases

#### 1. Authentication Data

```
Auth Middleware                              Handler
    │                                           │
    │ Verify token → extract user ID, role      │
    │ Save to context: { userId, role }         │
    │ next() ────────────────────────────────→   │
    │                                           │ Read userId from context
    │                                           │ Insert book with userId from context
    │                                           │ (NOT from client payload — security risk)
```

> **Always take user ID from the auth context, never from the client payload.** A malicious client could send another user's ID to perform unauthorized operations.

#### 2. Request ID for Tracing

An early middleware generates a UUID and saves it to context. Every downstream layer uses this ID for:
- Logging (correlate all logs for one request)
- External API calls (pass as `X-Request-ID` header)
- Debugging in microservice architectures (trace a request across services)

#### 3. Cancellation Signals and Deadlines

Context can carry abort signals and timeouts to prevent downstream services from hanging indefinitely — if the client disconnects or a deadline passes, downstream operations get cancelled.

---

## Summary

| Component | Responsibility | Deals with |
|-----------|---------------|-----------|
| **Handler/Controller** | HTTP concerns — binding, validation, response codes | Request & response objects |
| **Service** | Business logic — orchestration, external calls, notifications | Pure data processing |
| **Repository** | Database operations — queries, inserts, deletes | Database only |
| **Middleware** | Cross-cutting concerns — auth, logging, CORS, error handling | Runs between boundaries |
| **Request Context** | Shared state per request — user ID, role, request ID, deadlines | Accessible everywhere in the request lifecycle |

### Key Principles

- **Controller** = HTTP boundary. **Service** = business logic. **Repository** = database. Keep them separated.
- Service methods should look like plain functions — no HTTP awareness
- Repository methods have **single responsibility** — one method, one operation
- Middlewares reduce duplication for operations common to all/most requests
- Middleware **order matters** — CORS first, error handling last
- Request context decouples middlewares from each other — shared state without tight coupling
- **Never trust client-provided user IDs** — always use the authenticated identity from context
