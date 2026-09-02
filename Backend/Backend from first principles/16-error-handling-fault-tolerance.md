# Table-of-Contents

<!-- toc -->

- [Error Handling and Fault-Tolerant Systems](#error-handling-and-fault-tolerant-systems)
  * [The Mindset](#the-mindset)
  * [Types of Errors](#types-of-errors)
    + [1. Logic Errors](#1-logic-errors)
    + [2. Database Errors](#2-database-errors)
    + [3. External Service Errors](#3-external-service-errors)
    + [4. Input Validation Errors](#4-input-validation-errors)
    + [5. Configuration Errors](#5-configuration-errors)
  * [Prevention — Proactive Error Detection](#prevention--proactive-error-detection)
    + [Health Checks](#health-checks)
  * [Error Recovery Philosophies](#error-recovery-philosophies)
    + [Recoverable vs Non-Recoverable Errors](#recoverable-vs-non-recoverable-errors)
    + [Error Recovery Strategies](#error-recovery-strategies)
    + [Propagation Control (Error Bubbling)](#propagation-control-error-bubbling)
  * [Global Error Handling — The Final Safety Net](#global-error-handling--the-final-safety-net)
    + [How It Works](#how-it-works)
    + [Error Type → Response Mapping](#error-type-%E2%86%92-response-mapping)
    + [Two Key Advantages](#two-key-advantages)
  * [Security Considerations in Error Handling](#security-considerations-in-error-handling)
    + [1. Never Expose Internal Details](#1-never-expose-internal-details)
    + [2. Generic Auth Error Messages](#2-generic-auth-error-messages)
    + [3. Never Log Sensitive Data](#3-never-log-sensitive-data)
  * [Summary](#summary)

<!-- tocstop -->

---

# Error Handling and Fault-Tolerant Systems

**Source:** Sriniously — Backend from First Principles (Video 16)
**Link:** [Watch](https://www.youtube.com/watch?v=estH64OkwxU)

---

## The Mindset

Errors are not exceptional — they are **normal**. The question is not whether errors will happen but how you handle them when they do.

A fault-tolerant mindset means:
- Being prepared for the worst
- Detecting errors before they cause damage
- Having a plan for recovery

> **The best error handling starts before the error happens.**

---

## Types of Errors

### 1. Logic Errors

The most dangerous type — they don't crash your app, they make it **do the wrong thing silently**.

- Code runs fine, but results are incorrect
- Can go unnoticed for weeks or months
- Quietly cause real-world damage (financial loss, data corruption, wrong business outcomes)

**Common causes:**
- Misunderstood requirements
- Incorrectly implemented algorithms
- Unanticipated edge cases in business logic

> **Example:** An e-commerce store accidentally applies a discount twice → negative shipping cost → platform loses money on every order.

### 2. Database Errors

Can bring down the entire system since most backends depend heavily on their database.

| Type | Cause | Effect |
|------|-------|--------|
| **Connection errors** | DB server overloaded, network down, connection pool exhausted | App cannot function at all |
| **Constraint violation** | Inserting duplicate email (unique constraint), referencing non-existent foreign key | Bubbles up as 500 if not handled |
| **Query errors** | Malformed SQL, typo in table name, overly complex queries timing out | Query fails at execution |
| **Deadlocks** | Multiple operations waiting for each other in circular dependency | Operations stall indefinitely |

### 3. External Service Errors

Every external dependency is a point of failure you don't control.

| Type | Examples | Strategy |
|------|---------|---------|
| **Network errors** | Connection timeouts, DNS failures, network partitions | Retry with exponential backoff |
| **Auth errors** | Bad credentials, expired tokens, insufficient permissions | Validate config at startup |
| **Rate limiting** | Hitting `429 Too Many Requests` | Exponential backoff, reduce request frequency |
| **Service outage** | Cloud provider incident, maintenance window | Fallbacks, in-memory backup, secondary node |

**Rate limiting strategy — exponential backoff:**
- `429` received → wait 1 min → retry
- Still `429` → wait 2 min → retry
- Still `429` → wait 4 min → retry
- ...until successful response

### 4. Input Validation Errors

Caused by users sending data that doesn't meet your system's rules. Your **first line of defense**.

| Type | Example |
|------|---------|
| **Format validation** | Email not a valid email format, phone not a valid number |
| **Range validation** | String too long/short, number too high/low, array too large/small |
| **Required field** | Mandatory field missing from payload |

These are the easiest errors to handle — you already know your requirements and enforce them at the entry point. Always return `400 Bad Request` with a clear message.

### 5. Configuration Errors

Happen when moving between development, staging, and production environments.

**Common scenario:** Added a new environment variable in `.env` locally, forgot to add it to the production secret store → new deployment misbehaves.

**Two outcomes:**
1. **Best case:** App validates all required env vars at startup → fails to start → previous deployment continues serving (blue-green deployment pattern)
2. **Worst case:** App starts without validation → crashes at runtime when the missing variable is accessed → users get `500`

> **Best practice:** Validate all required configuration variables before your server starts. Fail fast with a meaningful message rather than failing silently in production.

---

## Prevention — Proactive Error Detection

### Health Checks

**HTTP health check:** Expose a `/health` or `/status` endpoint that returns `200 OK` when the service is running.

**Database health check:** Don't just check connectivity — run a representative query and measure its performance. Sudden query slowdown is an early warning sign.

**External service health checks:**

| Service type | How to verify |
|-------------|--------------|
| Payment processors | Run test transactions |
| Email services | Send test messages to internal email |
| Auth providers | Generate and validate test tokens |

**Core functionality checks:**
- All required configuration variables are loaded
- Required caches are populated
- Critical internal data structures are consistent

---

## Error Recovery Philosophies

### Recoverable vs Non-Recoverable Errors

| Error type | Strategy |
|-----------|---------|
| **Recoverable** (network blip, temp resource exhaustion) | Retry with exponential backoff |
| **Non-recoverable** (corrupted state, permanent service loss) | Graceful degradation — serve cached data, disable non-essential features, provide fallback |

> When implementing retry/backoff logic, make sure it doesn't add more load to an already stressed system.

### Error Recovery Strategies

**Automatic recovery:**
- Restart failed services automatically
- Clean up corrupted caches
- Switch to backup systems

**Manual recovery (requires human judgment):**
- Document recovery runbooks
- Test them ahead of time so they can be executed quickly under stress
- **Prioritize data integrity** — take backups at key moments, use transaction rollbacks, replay transaction logs

### Propagation Control (Error Bubbling)

Not every error should be handled at the layer where it occurs. Errors should **bubble up** with context to a higher-level handler.

**Pattern in try/catch languages (JS, Python):**
- Catch lower-level exceptions
- Wrap with business context
- Re-throw to propagate up to the global error handler

**Pattern in Go:**
- Return errors from repository → service → handler → middleware

**Error boundaries in service architecture:**
- Separate services into different processes
- Implement timeouts between services
- Use message queues (RabbitMQ, SQS) for async communication — a bug in one service doesn't take down another

---

## Global Error Handling — The Final Safety Net

A centralized middleware layer that catches **all errors** from all layers and converts them into appropriate HTTP responses.

### How It Works

```
Request → Router → Handler → Service → Repository
                                           │
                               (error occurs, bubbles up)
                                           │
                    ←──────────────────────┘
                    Global Error Handler Middleware
                           │
                    Inspect error type → return appropriate response
```

### Error Type → Response Mapping

| Error type | HTTP response |
|-----------|--------------|
| Validation failure | `400 Bad Request` + field-level error messages |
| Unique constraint violation | `400 Bad Request` + "resource already exists" |
| No rows returned (resource not found) | `404 Not Found` + "resource with ID X does not exist" |
| Foreign key violation | `404 Not Found` + "referenced resource does not exist" |
| Unknown / unhandled error | `500 Internal Server Error` + generic "something went wrong" |

### Two Key Advantages

1. **Robustness** — no error slips through as a raw `500` with internal details leaking to the user. All known error types are handled explicitly.
2. **No redundancy** — without a central handler, every repository method would need to duplicate the same error type-checking logic. One central place handles it all.

---

## Security Considerations in Error Handling

### 1. Never Expose Internal Details

Raw database errors often contain table names, constraint names, index names — valuable information for attackers crafting SQL injection attacks.

**Rule:** In your global error handler, always generate user-facing messages yourself. Never pass raw error messages from lower layers directly to the client.

**Default fallback:** If you don't know what the error is → `500` + `"Something went wrong"`. Never include raw stack traces or DB error strings.

### 2. Generic Auth Error Messages

Auth endpoints are the most targeted surface for attackers. Specific error messages enable **user enumeration attacks**:

| Bad (specific) | Attack enabled |
|---------------|---------------|
| "User with this email does not exist" | Attacker knows email is not registered → moves on |
| "Incorrect password" | Attacker knows email IS registered → brute forces password |

**Always return the same generic message for any auth failure:**

```
"Invalid email or password"
```

This prevents attackers from using your error messages as an oracle to discover valid accounts.

> Reference: [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)

### 3. Never Log Sensitive Data

Logs frequently end up in external log management services (ELK stack, Datadog, etc.). Data breaches often happen through leaked logs.

**Never log:**
- User emails or passwords
- Credit card numbers
- API keys or secrets
- Session tokens or JWTs

**Instead log:**
- User ID (not email)
- Correlation/request ID
- Error type and stack trace (internal logs only, not sent to client)

---

## Summary

| Concern | Key practice |
|---------|-------------|
| **Logic errors** | Testing, code review, monitoring business metrics |
| **Database errors** | Proper error handling per error type, connection pooling |
| **External service errors** | Expect failure, retry with backoff, implement fallbacks |
| **Validation errors** | Robust validation layer, return `400` with clear messages |
| **Configuration errors** | Validate all required vars at startup, fail fast |
| **Global error handling** | Central middleware, typed error responses, no raw internals |
| **Auth error security** | Generic messages to prevent user enumeration |
| **Log security** | Never log sensitive user data |
