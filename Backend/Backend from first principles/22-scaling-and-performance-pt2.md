# Table-of-Contents

<!-- toc -->

- [Scaling and Performance — Part 2](#scaling-and-performance--part-2)
  * [Statelessness — The Key Enabler of Horizontal Scaling](#statelessness--the-key-enabler-of-horizontal-scaling)
    + [Why Stateful Servers Break Horizontal Scaling](#why-stateful-servers-break-horizontal-scaling)
    + [Solutions: Externalize Everything](#solutions-externalize-everything)
  * [Load Balancers](#load-balancers)
    + [Load Balancing Algorithms](#load-balancing-algorithms)
    + [Health Checks](#health-checks)
  * [Database Scaling](#database-scaling)
    + [Read Replicas](#read-replicas)
    + [Sharding (Partitioning)](#sharding-partitioning)
  * [CDNs — Content Delivery Networks](#cdns--content-delivery-networks)
    + [The Physics Problem](#the-physics-problem)
    + [What CDNs Do](#what-cdns-do)
    + [What to Cache in CDNs](#what-to-cache-in-cdns)
    + [CDN as DDoS Protection](#cdn-as-ddos-protection)
    + [Edge Computing](#edge-computing)
  * [Asynchronous Processing](#asynchronous-processing)
    + [Synchronous vs Asynchronous Behavior](#synchronous-vs-asynchronous-behavior)
    + [The Pattern — Queue-Based](#the-pattern--queue-based)
    + [Good Candidates for Async Processing](#good-candidates-for-async-processing)
  * [Microservices vs Monolith](#microservices-vs-monolith)
    + [Monolith](#monolith)
    + [Microservices](#microservices)
  * [Serverless](#serverless)
    + [The Traditional Server Problem](#the-traditional-server-problem)
    + [The Serverless Model](#the-serverless-model)
    + [The Cold Start Problem](#the-cold-start-problem)
    + [Other Serverless Constraints](#other-serverless-constraints)
    + [When to Use Serverless](#when-to-use-serverless)
  * [Mental Models for Scaling and Performance](#mental-models-for-scaling-and-performance)
    + [1. Always Start with the Problem](#1-always-start-with-the-problem)
    + [2. Prefer Simple Solutions](#2-prefer-simple-solutions)
    + [3. Scale for the Problems You Have](#3-scale-for-the-problems-you-have)
    + [4. Measure from Day One](#4-measure-from-day-one)
    + [5. Performance is a Mindset](#5-performance-is-a-mindset)

<!-- tocstop -->

---

# Scaling and Performance — Part 2

**Source:** Sriniously — Backend from First Principles (Video 22)
**Link:** [Watch](https://www.youtube.com/watch?v=estH64OkwxU)

> Continuation of Video 21. Covers horizontal scaling requirements, load balancers, database scaling, CDNs, async processing, microservices, and serverless.

---

## Statelessness — The Key Enabler of Horizontal Scaling

Horizontal scaling (multiple server instances) only works if **no instance holds data exclusive to itself**. This property is called **statelessness**.

**The rule:** It must not matter which server handles a request. The result must always be the same. Any state that needs to persist must live *outside* all server instances, accessible by all of them.

### Why Stateful Servers Break Horizontal Scaling

If a user authenticates with instance A and A stores the session in its own memory, instance B and C have no access to that session. The next request routed to B returns `401 Unauthorized` — a confusing, broken experience.

### Solutions: Externalize Everything

| State type | Wrong approach | Correct approach |
|-----------|----------------|-----------------|
| Sessions | In-memory array in server instance | Redis (accessible by all instances) |
| File uploads | Local SSD of server instance | Object storage (S3, Cloudflare R2) |
| Database | SQLite file local to the server | Centralized Postgres / RDS |
| Local cache | In-process map | Redis or Memcached |

**The thumb rule:** If you choose horizontal scaling, at every point in your code, ask: "Is this data or file going to be stored in a single instance?" If yes, externalize it.

---

## Load Balancers

A **load balancer** sits in front of all server instances and distributes incoming requests among them. It is a mandatory component of horizontal scaling.

```
Users → Internet → Load Balancer → [Instance A, Instance B, Instance C]
                  (decides which instance gets the request)
```

### Load Balancing Algorithms

**Round-robin:** Sends requests in rotating order (A → B → C → A → ...). Simple and effective when requests are similar in cost and servers have equal capacity.

**Weighted round-robin:** Like round-robin, but sends proportionally more requests to higher-capacity servers. Example: if server A has 8GB RAM / 4 cores and B/C have 4GB / 2 cores, A gets 2× the requests.

**Least connections:** Routes each new request to the server with the fewest active connections at that moment. More intelligent — naturally avoids sending requests to overloaded servers handling expensive, long-running operations. (A connection remains active until the server sends a response.)

**Weighted least connections:** Combines least-connections logic with server capacity weighting.

**Other algorithms:** Least response time (favor faster-responding servers), resource-based (favor servers with lower CPU/memory usage).

### Health Checks

The load balancer continuously sends test requests (e.g., `GET /health` every second) to all instances. If an instance fails to return a `200` response, the load balancer:
1. Stops routing user requests to that instance
2. Keeps sending health check requests
3. Resumes routing once the instance recovers and returns `200`

This automatic detection of failed instances prevents users from hitting dead servers.

---

## Database Scaling

Application code scales cleanly by externalizing state. Databases are the **stateful component** — they cannot be naively duplicated. Two well-established patterns address this:

### Read Replicas

**Architecture:** One **primary** instance handles all writes (INSERT, UPDATE, DELETE). Multiple **replica** instances (read-only copies) handle SELECT queries. Replicas are kept in sync via a replication mechanism built into the database.

**Benefits:**
- Primary handles only ~30% of traffic (write operations); replicas absorb the ~70% that are reads
- Replicas can be placed in different geographic regions, reducing latency for users in those areas

**The consistency problem — replication lag:**
When a user writes data (e.g., updates their name from A to B on the US primary) and immediately reads it back (routed to an India replica), the replica may not yet have the updated data — the replication hasn't completed. The user sees stale data.

**Solutions to replication lag:**

| Solution | How it works |
|----------|-------------|
| Route reads after writes to primary | After a write, temporarily send all reads for that entity to the primary until replication completes |
| Track replication lag | Measure average lag (~200ms); hold read queries until replication is confirmed complete |
| Frontend delay | Wait ~300ms before issuing the follow-up GET request |

Managed database providers (AWS RDS, Google Cloud SQL) handle most replication complexity and expose simple configuration UI for setting up replicas in specific regions.

### Sharding (Partitioning)

**The problem:** A table with billions of rows (e.g., an orders table for a large e-commerce platform) has slow queries even with indexes, and a single instance can only handle so many requests/second.

**Sharding:** Physically divide one large table across multiple separate database instances, each holding a subset of the rows.

```
Orders table (10 billion rows)
  → Shard 1 (Jan–Jun):  instance 1   — 5 billion rows
  → Shard 2 (Jul–Dec):  instance 2   — 5 billion rows
```

The **shard key** determines how data is divided (in this example: order date). Your backend's routing layer determines which shard to query before making the database call.

**Benefits:**
- Each shard has fewer rows → faster queries
- Multiple shards handle more requests/second

**Modern option — distributed databases:** PlanetScale (MySQL-based), Neon (serverless Postgres), CockroachDB, YugabyteDB — these handle sharding, replication, backups, and distributed transactions automatically. As a backend engineer, you configure them via UI/settings rather than managing infrastructure yourself.

> Unless you have deep database administration expertise, always use a managed database provider. Understand these concepts so you can configure them correctly — not so you can implement them from scratch.

---

## CDNs — Content Delivery Networks

### The Physics Problem

Light travels through fiber optic cables at ~200,000 km/s. A request from Tokyo to a server in US East (North Virginia) — ~20,000 km round trip — has a minimum latency of **100ms** due to physics alone. No optimization can beat this.

Adding typical backend processing (deserialization, DB query, external API calls), the total latency for a Tokyo user becomes 500–800ms even with a well-optimized backend.

### What CDNs Do

CDNs place **edge nodes** (also called Points of Presence / PoPs) near users — in Tokyo, Mumbai, Singapore, etc. Instead of traveling to the US, the request hits the nearest edge node.

Tokyo → Local CDN node: ~100–200 km → **2–3ms latency** instead of 100ms.

**Two wins:**
1. **Lower latency** for users
2. **Reduced load** on your primary server — CDN nodes serve cached content directly, traffic doesn't reach the origin

### What to Cache in CDNs

| Content type | Examples | Notes |
|-------------|---------|-------|
| **Static assets** | JS/CSS bundles, HTML, images, videos, fonts | Best use case; changes infrequently; how SPAs are deployed |
| **API responses** | Product catalogs, public content | Cache with TTL; purge/tag-based invalidation when data changes |

### CDN as DDoS Protection

Cloudflare's CDN sits in front of your server. During a DDoS attack (thousands of bots sending traffic), Cloudflare's global network absorbs the traffic volume and applies rate limiting, CAPTCHAs, and bot detection — your origin server is shielded.

### Edge Computing

Traditional CDNs only serve cached static files. **Edge computing** adds processing at the edge node — code runs at the CDN layer before reaching your primary server.

**Use cases:**
| Use case | Benefit |
|----------|---------|
| **Authentication** | Reject invalid sessions at the edge in 2–3ms instead of sending them 100ms to the origin for a 401 |
| **Geo-based customization** | Serve localized content (language, currency) based on the user's region, decided at the edge |
| **Routing/validation** | Route requests, validate inputs before they reach the origin |

**Edge computing platforms:** Cloudflare Workers (V8 isolates), AWS Lambda@Edge.

**Constraints:** Edge nodes have limited RAM (~1GB) and limited runtimes (no file system access, no TCP). They cannot replace primary servers — they complement them by handling lightweight, latency-sensitive logic at the boundary.

---

## Asynchronous Processing

Some operations do not require the user to see the result immediately. Offloading these to background processing reduces **perceived latency** without changing the actual outcome.

### Synchronous vs Asynchronous Behavior

**Synchronous (traditional HTTP):** User waits for the full operation to complete before seeing a response.

**Asynchronous:** Server does minimal processing, returns success immediately, then a background worker completes the rest.

### The Pattern — Queue-Based

```
Request arrives
  → Server does minimal processing (validate, save minimal record)
  → Push task to queue
  → Return 200 to user immediately

Background worker:
  → Picks task from queue
  → Executes the expensive operation
  → (User doesn't wait for this)
```

**Queue options:** Redis Queue, BullMQ (Node.js, uses Redis), RabbitMQ, Kafka.

Workers (consumers) can run in the same codebase as your server or as a separate, independently scalable service.

### Good Candidates for Async Processing

| Operation | Why async works |
|-----------|----------------|
| **Sending emails/notifications** | User doesn't expect instant delivery; outcome is eventually visible |
| **Invite flows** | Validate + save invite record synchronously (100ms); send invite email asynchronously |
| **Video/image processing** | User uploads → server queues processing jobs (generate thumbnails, encode video, generate subtitles) |
| **Account deletion** | Delete session record synchronously → return 200 + log user out; delete all user data from 8+ tables async |
| **Report generation** | Queue, process, notify user when done |

**Concrete example — account deletion:** Deleting a user with 1M to-dos across 8 tables takes ~4s synchronously. Asynchronously: validate + queue the deletion job (100ms) → return 200 → background worker runs all delete queries over time.

> Async processing is one of the few optimizations worth implementing from day one, not just at scale.

---

## Microservices vs Monolith

### Monolith

A **monolith** is a single deployable unit — all functionality (auth, orders, payments, notifications) lives in one codebase, deployed as one process.

**Advantages:** Simple to develop, test, deploy, and refactor. Everything in one place.

**When monoliths become difficult** (with large teams):

| Problem | Description |
|---------|-------------|
| **Deployment dependency** | Module A wants to deploy; Module B has unfinished changes in the same main branch — can't deploy independently |
| **Scaling coupling** | Can't scale just the payments module; must scale everything, even notification which uses far fewer resources |
| **Technology lock-in** | Can't use Go for CPU-heavy image processing and Node.js for markdown parsing in the same deployable unit |

### Microservices

Divide the application into **independently deployable services**, each with a clear boundary.

**Advantages:**
- Deploy, scale, and monitor each service independently
- Use different technology stacks per service
- Teams can work autonomously with clear boundaries

**Disadvantages:**

| Complexity | Detail |
|-----------|--------|
| **Network calls** | What was a function call is now an HTTP/gRPC call — adds latency, failure modes, timeouts, retries |
| **Debugging** | A single request may touch 4 services; must trace across all of them simultaneously |
| **Data consistency** | Each service often has its own database; cross-service consistency becomes a distributed systems problem |

**When microservices make sense:**
- Large teams (100+ developers) where deployment independence matters organizationally
- Modules with genuinely different scaling needs
- Modules that need different technology stacks

> Microservices are primarily about **scaling teams**, not scaling machines. Until you hit the organizational limits of a monolith, a monolith is almost always the right choice.

---

## Serverless

### The Traditional Server Problem

Traditional servers (VMs) are always on — you pay 24/7 regardless of traffic. This creates two problems:

| Problem | Effect |
|---------|--------|
| **Underprovisioning** | Traffic spike crashes the server; users see errors |
| **Overprovisioning** | Paying for unused capacity |

**Autoscaling** mitigates this but has its own issues:
- Spinning up a new VM takes seconds to minutes (boot OS → install app → configure networking)
- Reactive, not proactive — new instances start only after overload is detected
- Always-on minimum (you still pay for baseline instances)

### The Serverless Model

You provide only **functions + triggering events**. The provider handles all infrastructure.

```
Event (HTTP request) → API Gateway → Routes to function → Spins up instance → Executes → Returns response → Instance released
```

**Pricing:** Pay only for actual execution time (CPU time per millisecond), not 24/7 uptime.

**Key benefit:** Scales from zero to thousands of requests automatically, no capacity planning required.

### The Cold Start Problem

When no instance is warm, the first request triggers spinning up a new container/VM:
1. Boot OS or runtime
2. Load your code

This startup time is the **cold start**. It's the primary trade-off of serverless.

**Solutions to cold start:**
- **Keep-warm pings:** Automated requests every few seconds keep instances alive (partially defeats the cost benefit)
- **Firecracker microVMs** (AWS Lambda): Lightweight VM technology; boots in milliseconds vs. seconds for full VMs
- **V8 isolates** (Cloudflare Workers): JavaScript runtime sandboxes that boot in 0–1ms; combined with JS (interpreted, no compile step) — cold starts are ~5ms

### Other Serverless Constraints

| Constraint | Detail |
|-----------|--------|
| **Execution limits** | AWS Lambda: max 15 minutes per invocation; incompatible with long-running operations |
| **Statelessness** | No persistent TCP connections, no file system; architecture must be redesigned |
| **Runtime limits** | No raw TCP, limited file system access in most edge runtimes |

### When to Use Serverless

**Good fit:**
- Event-driven pipelines (new file uploaded → trigger processing)
- Infrequent heavy tasks (video encoding, image resizing)
- Webhooks, scheduled jobs, data pipelines
- Edge authentication / routing logic

**Poor fit:**
- Latency-sensitive user-facing APIs (cold starts unpredictable)
- Long-running operations
- Applications requiring many persistent database connections
- Workloads with constant, predictable traffic (always-on servers are more cost-effective)

> Serverless is overhyped as a universal replacement for servers. It's a powerful tool for specific event-driven, bursty workloads — not a default architecture.

---

## Mental Models for Scaling and Performance

### 1. Always Start with the Problem

All scaling techniques are solutions. Before reaching for solutions, measure your system:
- Use distributed tracing to find which component is slow
- Use `EXPLAIN ANALYZE` to find which database queries need indexes
- Use metrics (Prometheus + Grafana, or New Relic) to track utilization and latency

**Never guess which component is your bottleneck.** Fixing the wrong bottleneck wastes time and gives a false sense of improvement while the real problem remains.

### 2. Prefer Simple Solutions

Complexity has costs — every added component is another thing that can fail, must be monitored, and must be understood by your team.

| Complex solution | Simple alternative |
|----------------|-------------------|
| Microservices | Monolith (until team/scale forces otherwise) |
| Kubernetes + horizontal scaling from day one | Large vertical instance with autoscaling |
| Redis cache in front of all queries | Proper database indexes |

Only accept complexity when simplicity is genuinely insufficient.

### 3. Scale for the Problems You Have

Don't build for a million users on day one. Build for your current scale with reasonable headroom. As you grow, your observability will show you exactly where bottlenecks emerge. Generic performance advice from Netflix or Google blog posts may not apply to your specific workload.

### 4. Measure from Day One

Observability (logs, metrics, traces) is the one exception to "start simple." Implementing production-grade observability from the beginning always pays off:
- You detect problems before users do
- You know exactly where to optimize when needed
- You have historical data to understand trends

### 5. Performance is a Mindset

Performance optimization is an ongoing discipline, not a one-time implementation. It comes from building systems, watching them struggle under real traffic, measuring the results, and iterating. The skill is knowing how to measure, diagnose, and resolve problems quickly — not predicting every possible failure in advance.
