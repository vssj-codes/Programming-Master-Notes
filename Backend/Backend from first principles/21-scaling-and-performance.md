# Table-of-Contents

<!-- toc -->

- [Scaling and Performance](#scaling-and-performance)
  * [What Does "Fast" Actually Mean?](#what-does-fast-actually-mean)
  * [Measuring Performance: Percentiles, Not Averages](#measuring-performance-percentiles-not-averages)
    + [The Three Key Percentiles](#the-three-key-percentiles)
  * [Throughput](#throughput)
  * [Utilization and Latency](#utilization-and-latency)
    + [The Counterintuitive Relationship](#the-counterintuitive-relationship)
    + [The Practical Rule](#the-practical-rule)
  * [Identifying Bottlenecks — Measure, Never Guess](#identifying-bottlenecks--measure-never-guess)
    + [A Real Example](#a-real-example)
  * [Profiling and Distributed Tracing](#profiling-and-distributed-tracing)
    + [Profiling (CPU-bound tasks)](#profiling-cpu-bound-tasks)
    + [Distributed Tracing (IO-bound tasks)](#distributed-tracing-io-bound-tasks)
  * [Database Performance](#database-performance)
    + [1. The N+1 Query Problem](#1-the-n1-query-problem)
    + [2. Indexes](#2-indexes)
    + [3. Connection Pooling](#3-connection-pooling)
  * [Caching for Performance](#caching-for-performance)
    + [Cache Invalidation](#cache-invalidation)
    + [Local vs Distributed Caching](#local-vs-distributed-caching)
    + [Caching Patterns](#caching-patterns)
    + [Cache Hit Rate](#cache-hit-rate)
  * [Scaling Approaches](#scaling-approaches)
    + [Vertical Scaling (Scaling Up)](#vertical-scaling-scaling-up)
    + [Horizontal Scaling (Scaling Out)](#horizontal-scaling-scaling-out)
  * [Summary](#summary)

<!-- tocstop -->

---

# Scaling and Performance

**Source:** Sriniously — Backend from First Principles (Video 21)
**Link:** [Watch](https://www.youtube.com/watch?v=estH64OkwxU)

---

## What Does "Fast" Actually Mean?

When a user clicks a button, a chain of events occurs: browser → internet → server → database → (external APIs) → response → browser renders. The total elapsed time from click to render is **latency**.

Latency is what users feel when they say an app is "slow" or "fast."

---

## Measuring Performance: Percentiles, Not Averages

**Averages are misleading for performance.** Consider: 1,000 requests measured, average latency = 100ms. But if 99% complete in 50ms and 1% take 5 seconds — the average hides a real problem affecting thousands of users.

### The Three Key Percentiles

| Metric | Meaning | Example |
|--------|---------|---------|
| **P50** (50th percentile) | 50% of requests are at or below this latency | P50 = 400ms → half your users wait 400ms |
| **P90** (90th percentile) | 10% of requests experience this latency or worse | P90 = 900ms → 10% of users wait 900ms |
| **P99** (99th percentile) | 1% of requests experience this latency or worse | P99 = 2s → 1% of users wait 2 seconds |

**Why P99 and P95 matter most:**
- Requests with the highest latency represent the most complex workflows (payments, multi-service calls, heavy queries)
- These users are often your highest-value customers — the ones making purchases or running complex operations
- Fixing P99 means fixing your most expensive code paths

---

## Throughput

**Throughput** = how many requests your system can handle per unit of time (requests/second or requests/minute).

A system can have impressive latency at low load (10 req/s → 150ms) but degrade badly at high load (1,000 req/s → 2,000ms). Throughput and latency together answer practical questions like:
- Can we survive a Black Friday traffic spike?
- How many concurrent users can we support before needing more resources?
- What happens if we run a large email campaign and suddenly get 10x traffic?

**Key insight:** As throughput increases, latency increases — slowly at first, then exponentially near capacity limits.

---

## Utilization and Latency

**Utilization** = the percentage of your system's capacity currently in use.

- 0% utilization → system is idle
- 100% utilization → system is maxed out, on the brink of collapse

### The Counterintuitive Relationship

We intuitively expect latency to grow linearly with utilization. It doesn't:

```
Latency
  │                          ╱
  │                        ╱
  │                      ╱
  │                   ╱
  │                ╱
  │           ╱
  │      ╱
  └──────────────────────────── Utilization
  0%    50%    80%   90%  100%
```

Near 100% utilization, latency grows **exponentially** — not linearly. This is because requests form a growing queue, and each new request waits longer for all the ones ahead of it.

### The Practical Rule

**Never run systems at 100% utilization.** Production systems typically target **60–80% utilization**, reserving 20–40% as a buffer for:
- Traffic bursts (traffic never arrives as a smooth, predictable stream — it always comes in spikes)
- Unexpected spikes from campaigns, press coverage, or seasonal events

---

## Identifying Bottlenecks — Measure, Never Guess

A **bottleneck** is the specific part of your system causing the slowness. The most common mistake is skipping this step and jumping straight to standard solutions (add caching, upgrade the database version, add more servers).

**The lesson:** Without measuring, you cannot know which component is adding milliseconds.

### A Real Example

An API endpoint appears slow. Assumption: database is the problem. Solution implemented: add Redis caching. Result: still slow.

Root cause (discovered after adding timing logs): a logging function was writing synchronously to a remote logging service — taking 500ms. The database query was only 10ms.

The cache was irrelevant. Spending a week implementing it changed nothing.

**Always measure first.** Add timing measurements at each component. Find the actual culprit before implementing solutions.

---

## Profiling and Distributed Tracing

### Profiling (CPU-bound tasks)

A **profiler** attaches to your running application and records which functions are executing, when, and for how long. Output is visualized as a **flame graph** — wider bars = more time spent.

Profilers are most useful for **CPU-bound tasks**: computation-heavy operations (image processing, ML inference, data transformation).

**Limitation:** Profilers are not great at measuring IO-bound tasks (database queries, external API calls, file handling). Most typical SaaS backend bottlenecks are IO-bound.

### Distributed Tracing (IO-bound tasks)

**Distributed tracing** follows a single request as it flows through your system and records timestamps at every component:

```
Request: GET /products/5

→ Entered API handler:         t=0ms
→ Started DB query:            t=2ms
→ DB query returned:           t=802ms   ← 800ms here!
→ JSON serialization:          t=803ms
→ Response sent:               t=805ms
```

This immediately shows you where to focus: the database query is the bottleneck, not the business logic, serialization, or anything else.

Distributed tracing is part of the **observability** stack (covered in Video 18). Tools: Jaeger, OpenTelemetry, New Relic, Datadog.

---

## Database Performance

Databases do the hard work: durable storage on disk, consistency guarantees for concurrent reads/writes, complex queries across millions of rows. Three common performance concerns:

### 1. The N+1 Query Problem

**The problem:** Fetching a list of N items, then making a separate query for each item to get related data.

```
// WRONG: N+1 pattern
posts = db.select(posts).where(...)         // 1 query
for post in posts:
    author = db.select(users).where(id=post.author_id)  // N queries
```

For 1,000 posts: 1,001 queries × 5ms each = **5 seconds**.

Each query has overhead: TCP connection (if not pooled), query parsing, execution planning, network round-trip.

**Fix: bulk fetch / joins**

```
// CORRECT: fetch all data in bulk
posts = db.select(posts).join(users).where(...)    // 1 query, all data

// Or: collect all IDs first, then fetch in one query
author_ids = [post.author_id for post in posts]
authors = db.select(users).where(id IN author_ids) // 2 queries total
```

Regardless of whether you show 20, 100, or 1,000 posts: always 2 queries.

> **ORM warning:** ORM code looks like normal language constructs, which hides N+1 patterns. Enable SQL query logging during development to see what queries your ORM actually runs. Use `select_related`, `prefetch_related`, `includes`, `joins` — whatever your ORM provides for bulk fetching.

### 2. Indexes

Without an index, a database performs a **full table scan (sequential scan)**: it reads every row to find matches. For a million rows: ~4 seconds. With an index: ~40–100ms.

**What an index is:** A B-tree data structure that maintains a sorted copy of a column's values with pointers to the actual rows. Enables lookups without scanning every row.

**The cost of indexes:**

| Cost | Detail |
|------|--------|
| **Storage** | Each index takes disk space proportional to table size |
| **Write overhead** | Every INSERT, UPDATE, DELETE must also update all indexes on that table |

> Don't index every column. Excessive indexes make writes significantly slower.

**When to add an index:**
- At migration time: obvious columns (foreign keys used in frequent JOINs, columns always in WHERE clauses)
- Primary keys are indexed by default (Postgres)
- After launch: use distributed tracing to find slow queries, then use `EXPLAIN ANALYZE` to confirm

**`EXPLAIN ANALYZE`** — runs a query and shows the full execution plan:
- Which tables were scanned (sequential scan = no index)
- Which indexes were used (index scan = good)
- Time spent at each step

Add the index, run `EXPLAIN ANALYZE` again, confirm it switched from sequential scan to index scan.

**Composite indexes:** An index on multiple columns `(user_id, created_at)`. Helps queries that filter on both. Also helps queries that filter on just `user_id` (leading column). Does NOT help queries filtering only on `created_at`.

**Covering indexes:** Includes all columns needed by a query. Database can serve the result entirely from the index — never touches the main table.

### 3. Connection Pooling

Each database connection requires:
1. TCP three-way handshake
2. Authentication
3. Encryption negotiation
4. Session state setup
5. Memory allocation in the database

Creating a new connection for every query is expensive. Databases also have connection limits (Postgres: ~400–500 by default).

**Connection pooling:** A pool maintains a set of pre-established, idle connections. When a query needs to run:
1. Borrow a connection from the pool
2. Execute the query
3. Return the connection to the pool (keep it open for reuse)

| Pool type | How it works | Problem |
|-----------|-------------|---------|
| **Internal** | Pool managed by each server instance's DB driver | Multiple server instances each have their own pool → can exceed DB connection limit during horizontal scaling |
| **External** | Standalone pool service (e.g., **PgBouncer** for Postgres) shared by all server instances | All instances share one pool → total connections stay bounded |

**In production with horizontal scaling: use an external pooler (PgBouncer).** Internal pooling works fine for a single server instance.

---

## Caching for Performance

The idea: store the result of expensive operations (slow DB queries) in fast storage (Redis). Next request gets the result from cache in ~5ms instead of running the expensive query again for ~800ms.

### Cache Invalidation

The hard problem. When the underlying data changes, the cache must be updated or cleared.

**Two strategies:**

| Strategy | How it works | Trade-off |
|----------|-------------|-----------|
| **Time-based (TTL)** | Cache expires automatically after a set duration | Simple; risk of serving stale data until TTL expires |
| **Event-based** | Delete/update cache entry immediately when data changes | Always fresh; must remember to invalidate at every write code path |

### Local vs Distributed Caching

| Type | What it is | Trade-off |
|------|-----------|-----------|
| **Local** | In-process data structure (map/dict) in server memory | ~2ms access; inconsistent across multiple server instances |
| **Distributed** | External service (Redis, Memcached, Valkey) | ~50ms network round-trip; consistent across all instances |

**Tiered caching:** Combine both — local cache for the "hottest" (most frequently accessed) data; distributed cache as the upstream fallback.

### Caching Patterns

| Pattern | How it works | Best for |
|---------|-------------|---------|
| **Cache-aside (lazy loading)** | Check cache → miss → query DB → populate cache → return. On write: invalidate cache entry. | Read-heavy workloads; most common pattern |
| **Write-through** | On every write: update DB and cache simultaneously before returning success | Never a cache miss; adds write latency |
| **Write-behind** | On write: update cache immediately, return success, update DB asynchronously | Lower write latency; risk of data loss if async DB write fails |

### Cache Hit Rate

**Cache hit rate** = percentage of requests served from cache without hitting the database.

A 90% hit rate is excellent. A 20% hit rate means your caching strategy needs work.

Factors affecting hit rate:
- **TTL**: longer TTL → more hits, more staleness risk
- **Cache size**: more memory → more data fits → more hits
- **Data access patterns**: if you don't understand which endpoints are frequently accessed, your caching strategy will be poorly targeted

---

## Scaling Approaches

### Vertical Scaling (Scaling Up)

Replace your server with a more powerful one: more CPU cores, more RAM, faster storage (NVMe SSD), faster network card.

**Advantages:**

| Advantage | Detail |
|-----------|--------|
| **Simplicity** | No code changes, no architectural changes |
| **Linear capacity gains** | 2× CPU → ~2× request capacity |
| **Economical** | One large server often costs less than two medium servers; no load balancer overhead |

**Limitations:**

| Limitation | Detail |
|-----------|--------|
| **Hard ceiling** | Cloud providers have a maximum instance size; eventually you can't scale further |
| **Single point of failure** | One server crashes → service completely unavailable until recovery |
| **No geographic distribution** | A server in the US has high latency for users in India; can't fix this with one machine |

### Horizontal Scaling (Scaling Out)

Add more instances of the same server that work together to serve traffic.

**Advantages:**

| Advantage | Detail |
|-----------|--------|
| **No hard limit** | Just keep adding instances |
| **Redundancy** | If one instance fails, others continue serving traffic |
| **Geographic distribution** | Deploy instances in multiple regions; route users to the nearest one |
| **Elastic scaling** | Add instances during traffic spikes, remove them after |

**Disadvantages — the complexity cost:**

| Challenge | What you now need |
|-----------|-----------------|
| Traffic distribution | A **load balancer** (new component, new complexity) |
| Load balancing algorithm | Round-robin? Least connections? Consistent hashing? Another decision |
| State synchronization | If user updates name on server 1, how does server 2 know? |
| Network partition handling | If servers can't communicate, they may make conflicting decisions |
| Health detection | How do you detect a failed instance and stop routing to it? |

Horizontal scaling doesn't eliminate problems — it transforms one set of problems (capacity limits) into another set (distributed systems complexity). The question is which set of problems is more favorable for your situation.

---

## Summary

| Concept | Key point |
|---------|----------|
| **Latency** | Total time from user action to response — what users feel as "fast" or "slow" |
| **Percentiles** | Use P50/P90/P99 — not averages; P99 reveals your most expensive workflows |
| **Throughput** | Requests per second; latency degrades exponentially near capacity |
| **Utilization** | Run at 60–80%, not 100%; reserve headroom for traffic bursts |
| **Bottleneck** | Always measure before guessing; use distributed tracing + `EXPLAIN ANALYZE` |
| **N+1 queries** | Never fetch related data in a loop; use joins or bulk fetches |
| **Indexes** | Dramatically speed reads; add cost to writes — index deliberately, not generously |
| **Connection pooling** | Reuse connections; use external pooler (PgBouncer) when horizontally scaled |
| **Cache invalidation** | The hard problem; choose TTL vs event-based based on staleness tolerance |
| **Caching patterns** | Cache-aside (most common), write-through (no misses), write-behind (lower write latency) |
| **Vertical scaling** | Simple, has hard limits and single point of failure |
| **Horizontal scaling** | Unlimited scale and redundancy, introduces distributed systems complexity |
