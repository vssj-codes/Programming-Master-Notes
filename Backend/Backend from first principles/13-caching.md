# Table-of-Contents

<!-- toc -->

- [Caching — The Secret Behind It All](#caching--the-secret-behind-it-all)
  * [What is Caching?](#what-is-caching)
  * [Real-World Examples](#real-world-examples)
    + [Google Search](#google-search)
    + [Netflix](#netflix)
    + [Twitter Trending Topics](#twitter-trending-topics)
  * [Three Levels of Caching](#three-levels-of-caching)
    + [1. Network / CDN Caching](#1-network--cdn-caching)
    + [2. Hardware Caches (CPU and Memory Hierarchy)](#2-hardware-caches-cpu-and-memory-hierarchy)
    + [3. In-Memory Databases (Application-Level Cache)](#3-in-memory-databases-application-level-cache)
  * [Caching Strategies](#caching-strategies)
    + [Lazy Caching (Cache-Aside)](#lazy-caching-cache-aside)
    + [Write-Through Caching](#write-through-caching)
  * [Cache Eviction Policies](#cache-eviction-policies)
  * [Backend Use Cases](#backend-use-cases)
    + [1. Database Query Caching](#1-database-query-caching)
    + [2. Session Storage](#2-session-storage)
    + [3. External API Response Caching](#3-external-api-response-caching)
    + [4. Rate Limiting](#4-rate-limiting)
  * [Summary](#summary)

<!-- tocstop -->

---

# Caching — The Secret Behind It All

**Source:** Sriniously — Backend from First Principles (Video 13)
**Link:** [Watch](https://www.youtube.com/watch?v=estH64OkwxU)

---

## What is Caching?

**Caching** = storing data in a location where it can be retrieved faster than fetching it from the original source.

The core trade-off: **speed vs freshness**. Cached data is fast but may be stale.

---

## Real-World Examples

### Google Search
When you search for something popular ("Taylor Swift"), Google doesn't query millions of sources in real-time. The results are precomputed and cached — response is nearly instant.

### Netflix
Netflix doesn't stream video from a central server to every viewer. They use a **CDN (Content Delivery Network)** — copies of popular content cached on servers distributed globally. A viewer in India pulls from a nearby edge server, not from US data centers.

### Twitter Trending Topics
Recomputing trending topics from billions of tweets per request would be too expensive. Instead, the trending list is computed periodically and **cached** — served instantly to every user without touching the database.

---

## Three Levels of Caching

### 1. Network / CDN Caching

**CDN (Content Delivery Network):** A distributed network of servers geographically close to end users that cache static assets (images, videos, HTML, JS, CSS).

- Request from user → nearest CDN edge server → serves cached content
- Cache miss → CDN fetches from origin server → caches for future requests
- Dramatically reduces latency and load on origin servers

**DNS Caching:** DNS resolution maps domain names to IP addresses. This mapping is cached at multiple levels:
- Browser cache
- OS cache
- ISP / recursive resolver cache

On cache miss → DNS resolver queries authoritative DNS servers → result is cached with a TTL.

### 2. Hardware Caches (CPU and Memory Hierarchy)

| Cache | Size | Speed | Closest to |
|-------|------|-------|-----------|
| **L1 Cache** | ~32–64 KB | Fastest | Single CPU core |
| **L2 Cache** | ~256 KB – 1 MB | Fast | Single or shared core |
| **L3 Cache** | ~4–64 MB | Moderate | Shared across cores |
| **RAM** | GBs | Slower than CPU caches | Main memory |
| **Disk (SSD/HDD)** | TBs | Slowest | Persistent storage |

Each level is faster but smaller than the one below it. CPU checks L1 → L2 → L3 → RAM → disk.

### 3. In-Memory Databases (Application-Level Cache)

For backend engineers, this is the primary caching layer.

**Examples:** Redis, Memcached

**Why they're fast:**
- Data lives entirely in **RAM** (not disk)
- RAM access: ~100 nanoseconds vs disk access: ~100 microseconds+ — orders of magnitude faster

**RAM vs Disk trade-off:**

| Property | RAM | Disk |
|----------|-----|------|
| Speed | Very fast | Slow |
| Capacity | Limited (GBs) | Large (TBs) |
| Cost | Expensive | Cheap |
| Persistence | Volatile (data lost on restart) | Durable |

> Redis can be configured with persistence (AOF/RDB snapshots), but the primary advantage is in-memory speed.

---

## Caching Strategies

### Lazy Caching (Cache-Aside)

Cache is populated **on demand** — only when data is first requested.

**Flow:**
```
Client → Check cache
         ├── Cache HIT → return cached data
         └── Cache MISS → query database → store result in cache → return data
```

**Pros:**
- Cache only contains what's actually needed
- Simple to implement
- Works well for read-heavy workloads

**Cons:**
- First request always hits the database (cold start latency)
- Cache can become stale if database is updated directly

### Write-Through Caching

Cache is updated **synchronously** every time the database is written to.

**Flow:**
```
Write request → Update database AND update cache simultaneously
Read request  → Always hits cache (always up to date)
```

**Pros:**
- Cache is always consistent with the database
- No stale data

**Cons:**
- Write latency increases (two writes per operation)
- Cache may hold data that's never read (wasted memory)

---

## Cache Eviction Policies

When the cache is full, old entries must be removed to make room for new ones.

| Policy | How it works | Best for |
|--------|-------------|---------|
| **No Eviction** | Cache never evicts — returns error when full | Small, fixed datasets |
| **LRU** (Least Recently Used) | Evicts the item that hasn't been accessed the longest | General purpose, most common |
| **LFU** (Least Frequently Used) | Evicts the item accessed least often overall | Workloads with stable hot data |
| **TTL** (Time-To-Live) | Each entry has an expiry time — auto-evicted on expiry | Time-sensitive data (sessions, rate limits) |

> **LRU is the most common** default. Redis supports LRU, LFU, TTL, and others.

---

## Backend Use Cases

### 1. Database Query Caching

Expensive or frequently repeated DB queries are cached by their result.

```
GET /api/trending-posts

Check cache key "trending-posts"
  → HIT: return cached list (fast)
  → MISS: query DB → cache result with TTL → return list
```

When TTL expires or data changes, cache is invalidated and refreshed on next request.

### 2. Session Storage

User session data (user ID, role, auth status) is stored in Redis instead of a relational database.

- **Why:** Sessions are read on every authenticated request — must be fast
- **How:** Session ID (cookie) → Redis key lookup → user data returned in milliseconds
- **TTL** automatically expires sessions after inactivity

### 3. External API Response Caching

Responses from third-party APIs (e.g. weather, exchange rates, geolocation) are expensive or rate-limited.

```
GET /weather?city=London

Check cache key "weather:London"
  → HIT: return cached response (within TTL)
  → MISS: call external API → cache response → return data
```

Reduces latency, avoids rate limit violations, saves API quota costs.

### 4. Rate Limiting

Redis is ideal for tracking request counts per IP/user within a time window.

```
On each request:
  INCR counter for (user_id, current_minute)
  IF counter > limit → return 429 Too Many Requests
  SET TTL on key = 60 seconds (auto-resets each minute)
```

Atomic Redis operations (`INCR`) prevent race conditions even under high concurrency.

---

## Summary

| Layer | Technology | Speed | Typical Use |
|-------|-----------|-------|-------------|
| CDN | Cloudflare, Fastly, AWS CloudFront | Very fast | Static assets, video |
| DNS | OS / ISP cache | Fast | Domain resolution |
| CPU / RAM | L1–L3, RAM | Extremely fast | Hardware/OS-level |
| In-memory DB | Redis, Memcached | Fast | App-level caching |

| Strategy | When to use |
|----------|------------|
| **Lazy caching** | Read-heavy; acceptable cold-start latency |
| **Write-through** | Write-heavy; consistency is critical |

| Eviction | Best for |
|----------|---------|
| **LRU** | General purpose (default) |
| **LFU** | Stable hot data |
| **TTL** | Time-sensitive data |
