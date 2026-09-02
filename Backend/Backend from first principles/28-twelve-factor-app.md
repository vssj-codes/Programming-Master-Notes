# Table-of-Contents

<!-- toc -->

- [The Twelve-Factor App](#the-twelve-factor-app)
  * [What Is the Twelve-Factor App?](#what-is-the-twelve-factor-app)
    + [Historical Context — Software Erosion](#historical-context--software-erosion)
    + [The 2024 Open-Source Revision](#the-2024-open-source-revision)
  * [Factor 1 — Codebase](#factor-1--codebase)
  * [Factor 2 — Dependencies](#factor-2--dependencies)
    + [Declaration](#declaration)
    + [Isolation](#isolation)
  * [Factor 3 — Config](#factor-3--config)
    + [Why Environment Variables](#why-environment-variables)
    + [The Secrets Caveat](#the-secrets-caveat)
  * [Factor 4 — Backing Services](#factor-4--backing-services)
  * [Factor 5 — Build, Release, Run](#factor-5--build-release-run)
    + [Why Immutable Releases Enable Fast Rollback](#why-immutable-releases-enable-fast-rollback)
    + [Why Separation Prevents Drift](#why-separation-prevents-drift)
  * [Factor 6 — Processes](#factor-6--processes)
    + [What Processes Are Allowed to Hold](#what-processes-are-allowed-to-hold)
    + [Sticky Sessions = Violation](#sticky-sessions--violation)
    + [The WebSocket Exception](#the-websocket-exception)
  * [Factor 7 — Port Binding](#factor-7--port-binding)
  * [Factor 8 — Concurrency](#factor-8--concurrency)
    + [Process Types](#process-types)
    + [Process Formation](#process-formation)
  * [Factor 9 — Disposability](#factor-9--disposability)
    + [Fast Startup](#fast-startup)
    + [Graceful Shutdown (SIGTERM)](#graceful-shutdown-sigterm)
    + [Crash-Only Design](#crash-only-design)
  * [Factor 10 — Dev/Prod Parity](#factor-10--devprod-parity)
    + [The Tools Gap — A Dangerous Example](#the-tools-gap--a-dangerous-example)
    + [The Managed Services Tension](#the-managed-services-tension)
  * [Factor 11 — Logs](#factor-11--logs)
    + [The Rule](#the-rule)
    + [Why Streams Over Files](#why-streams-over-files)
  * [Factor 12 — Admin Processes](#factor-12--admin-processes)
    + [The Rule](#the-rule-1)
  * [Demo — Factors in Practice](#demo--factors-in-practice)
  * [Summary](#summary)

<!-- tocstop -->

---

# The Twelve-Factor App

**Source:** Sriniously — Backend from First Principles (Video 28)
**Link:** [Watch](https://www.youtube.com/watch?v=estH64OkwxU)

---

## What Is the Twelve-Factor App?

A **twelve-factor app** is a contract between your application and the platform that runs it. It states what must be true about your app so that any machine, any platform, and any new teammate can run it, scale it, and update it without surprises.

Written in 2011 by Adam Wiggins (Heroku co-founder) and published at [12factor.net](https://12factor.net), these rules were distilled from watching thousands of codebases get deployed, scaled, broken, and corrupted across many languages and frameworks.

Every modern deployment platform — Heroku, Docker, Kubernetes, Render, Railway, Fly.io, Cloud Run, App Runner, Vercel, Netlify — assumes these rules directly or indirectly.

**Scope:** Web apps and APIs that run as a service and are reachable over a network. Not desktop software or mobile apps.

**What the factors are NOT:** A complete list of good software practices (DRY, KISS, etc.), Kubernetes operations advice, or microservice architecture guidance.

### Historical Context — Software Erosion

Before cloud platforms, deploying an app meant:
1. Renting a server or using a physical machine on premises
2. Installing the OS by hand
3. Installing the language runtime by hand
4. Installing the database and supporting libraries by hand
5. Copying the application via FTP
6. Editing config files on the server via SSH

Every server built this way was called a **snowflake** — unique, hand-built, and impossible to reproduce exactly. If the machine died, you started from scratch. New teammates took days to set up. "It works on my machine" became a cultural phrase.

**Software erosion:** An app slowly stops working even though no code changed. Caused by OS patches, library conflicts, a departed team member who never documented their manual changes, or another app on the same server updating a shared dependency.

The 12 factors are defenses against erosion.

### The 2024 Open-Source Revision

For 13 years, the document was frozen (last updated 2017). In November 2024, Heroku open-sourced it under a neutral organization, invited maintainers from AWS, Google Cloud, and others, and a community rewrite (branch: `next`) is underway. The live site still shows the classic 2011 text. This note covers both the original and what the rewrite changes.

---

## Factor 1 — Codebase

> *One codebase tracked in revision control, many deploys.*

**The obvious part:** All code lives in version control (Git). In 2011 this still needed to be written down — production codebases were stored in folders named `final_v2_really_final.zip`.

**The non-obvious part — many deploys:**
A **deploy** = one running instance of your application.

| Deploy | Commit | Config |
|--------|--------|--------|
| Production | commit 97 (last Friday's release) | Production config |
| Staging | commit 99 (tomorrow's candidate) | Staging config |
| Local (laptop) | commit 100 + 2 unpushed | Local config |

Same codebase. Different deploys. The only thing that makes them different is **config** — not the code.

**Two prohibited patterns:**
- Multiple codebases for one app → that's a distributed system; each component is its own app
- Two apps sharing code by copy-paste → refactor into a library, use via dependency manager

**On monorepos:** A monorepo containing five apps is the same as five codebases in one repository. The factor restricts the same app living in two places, not multiple apps sharing a repo.

---

## Factor 2 — Dependencies

> *Explicitly declare and isolate dependencies.*

### Declaration

A manifest file lists every library the app needs to run, with pinned versions:

| Language | Manifest | Lock file |
|----------|---------|-----------|
| Node.js | `package.json` | `package-lock.json` |
| Go | `go.mod` | `go.sum` |
| Python | `requirements.txt` / `pyproject.toml` | (varies by tool) |
| Ruby | `Gemfile` | `Gemfile.lock` |

Without a manifest, setting up a new laptop or onboarding a new teammate requires reading every import statement and guessing compatible versions — a multi-day process. With a manifest: one command.

**Lock files** pin the exact version of every library and every library's library, so the same versions install months or years later.

### Isolation

Declaration alone breaks if the app uses the system's installed copy instead of the declared one.

```
# Python example of the problem:
Your requirements.txt says: requests==2.31
Your system has:             requests==1.2   (installed by some other tool)
Your app imports requests → gets version 1.2 → surprise behavior
```

**Fix:** Virtualenv (Python), `bundle exec` (Ruby). The running process sees only declared dependencies.

**Vendored tools:** If your app calls external binaries (e.g., ImageMagick for image resizing), those binaries must also be pinned. ImageMagick 6 (`convert`) and 7 (`magick`) have different CLI APIs — the same shell command behaves differently on macOS (Homebrew v7) vs. Ubuntu server (apt v6).

**Modern solution:** Docker images freeze not just libraries but the OS, runtime, and system tools — the peak of declaration and isolation. Docker practically solved this factor.

---

## Factor 3 — Config

> *Store config in the environment.*

**What counts as config:** Everything likely to differ between deploys — database URLs, API keys, email provider endpoints, Redis addresses, S3 bucket names, object storage endpoints.

**The open-source test:** Could you open-source your production codebase right now, at this moment, without leaking a single credential? If no, some config is sitting inside the code where it should not be.

### Why Environment Variables

| Reason | Detail |
|--------|--------|
| Easy to change | No file to edit; just set a variable |
| Cannot be accidentally committed | Credentials stay outside VCS |
| Language agnostic | Same mechanism for Go, Node, Python, bash |
| Framework agnostic | No proprietary config format |

**Anti-pattern — named environments:** `config/environments/staging.rb`, `production.rb`, `test.rb`. Every new deploy requires a new file. Config should be granular, individual variables — not environment name groups.

**Example — Tasker app:** 38 config values across server, database, auth, Redis, email, AWS, observability, cron. All loaded from environment variables with a `TASKER_` prefix. The config-reading code barely changed; the values changed constantly.

### The Secrets Caveat

Environment variables are appropriate for non-sensitive config. For high-security credentials (database passwords, API secrets):

- Environment can leak via child processes, crash reporters, debug pages, `/proc/<pid>/environ`
- **Correct pattern:** Use a secret manager (HashiCorp Vault, AWS Parameter Store, GCP Secret Manager). App reads credentials from the secret manager at boot, stores them in memory. Rotates and audits credentials.

> For solo or small team projects: platform-injected secrets via environment variables are acceptable and far better than committing credentials. At scale: always use a dedicated secret manager.

---

## Factor 4 — Backing Services

> *Treat backing services as attached resources.*

**Backing service:** Anything the app consumes over the network — Postgres, Redis, SMTP server, S3, payment provider APIs, third-party REST APIs.

**The rule:** Your code makes no distinction between a service you operate yourself and one operated by a third party. Both are attached resources, reachable through a **resource handle**.

**Resource handle = URL + credentials:**

```
postgres://user:password@host:5432/dbname
```

This single string gives the app everything it needs to connect. The only thing that changes between local and production is this string, stored in config (Factor 3). Not a line of code.

**The portability test:** You should be able to swap your local Postgres (Docker container) for Neon, PlanetScale, or Amazon RDS by changing one config value. Same for Redis → Upstash, local SMTP → Resend, local Minio → AWS S3.

**How it aged:** In 2011, server and database running on the same machine was still normal. Today, it's an anti-pattern. Managed backing services (RDS, Upstash, Resend) are the standard. This factor was essentially a prediction that came true.

---

## Factor 5 — Build, Release, Run

> *Strictly separate build and run stages.*

Three distinct stages:

```
Code (at commit N)
        │
        ▼ BUILD
Binary + dependencies
(no config included)
        │
        ▼ RELEASE = build + deployment config
Release #101   (immutable, append-only)
        │
        ▼ RUN
Running process(es)
```

| Stage | Input | Output | Properties |
|-------|-------|--------|-----------|
| **Build** | Source code at commit N | Executable bundle / Docker image | Immutable; no config included |
| **Release** | Build + deployment config | Numbered, timestamped release | Immutable; append-only ledger |
| **Run** | Release | Running process(es) | Can be started/stopped at any time |

### Why Immutable Releases Enable Fast Rollback

```
Release 100 → deployed Friday
Release 101 → config change (password rotation), same build
Release 102 → new code deployed → 405 errors appear

Action: rerun Release 101 → production back to stable in seconds
Then: debug Release 102 calmly, no users affected
```

Release 101 means exactly what it meant when first deployed — immutable.

### Why Separation Prevents Drift

When code only enters via the build stage, there is no path from the running machine back into the build. SSH hot-patching (`git pull` on the production server, reload server) is architecturally impossible. The production code and the repository cannot diverge.

**Modern mapping:** Docker image tags, Kubernetes rollout revisions, Heroku releases — all implement this model. The rewrite makes **immutable** explicit as a required principle.

---

## Factor 6 — Processes

> *Execute the app as one or more stateless processes.*

**Stateless = share nothing.** No shared memory, no shared disk between processes. Any request must be servable by any process. Any process can be destroyed at any moment.

Persistent state → backing services (Factor 4).

### What Processes Are Allowed to Hold

In-process memory and local disk are fine **within a single transaction** — e.g., downloading a file, processing it, writing the result to the database. The temp file is gone when the transaction ends. Nothing else depends on it existing.

### Sticky Sessions = Violation

**The problem:** App stores sessions in process memory. Three instances behind a load balancer. User logs into instance 1; next request routes to instance 2 → `401 Unauthorized`.

**The workaround (anti-pattern):** Sticky sessions — load balancer always routes a user to the same instance.

**Why it's wrong:**
- Defeats the purpose of multiple instances (if one dies, all its users are logged out)
- Adding a new instance doesn't help existing users

**The fix:** Session data is state → goes in a backing service (Redis, Postgres). All instances read from the same source.

### The WebSocket Exception

WebSocket connections are stateful by nature — they live in one specific process. This breaks strict statelessness. The mitigation (pub/sub across instances, covered in Video 26) acknowledges this and routes around it. The factor is the default; real-time features are a permitted, deliberate exception when the feature itself is the state.

**Thumb rule:** If you cannot kill any process at any moment without a user losing data, state is in the wrong place.

---

## Factor 7 — Port Binding

> *Export services via port binding.*

**Historical context:** In 2011, PHP apps ran as Apache modules. Java apps lived inside Tomcat. The web server's version and config were part of the application's runtime environment — part of the physical server. Erosion again.

**The rule:** The app is completely self-contained. It includes its own HTTP server as a library (from Factor 2's dependency manifest). It exports itself by binding to a port and listening for connections.

```
go build → ./server → binds :8080 → accepts connections
```

Development: `curl localhost:8080`
Production: reverse proxy routes public hostname → the port the process bound

**Consequence:** Any app can become a backing service for any other app — just point the resource handle at its port.

**What aged badly:** Serverless functions (AWS Lambda) don't bind to a port — the platform calls an exported function. The rewrite generalizes this to: *the app should be self-contained and export its service through a declared interface*, covering both long-running servers and serverless functions.

---

## Factor 8 — Concurrency

> *Scale out via the process model.*

**The rule:** Design for scaling by running more processes, not by making one process larger. Processes are first-class citizens.

### Process Types

| Type | Handles |
|------|---------|
| **web** | HTTP requests |
| **worker** | Background jobs (video encoding, email sending) |
| **scheduler** | Cron-like tasks (nightly emails, reports) |
| **clock** | Time-based triggers |

Each type scales **independently**:

```
# Food delivery app — lunchtime
web:    10  (high traffic)
worker:  2  (normal background load)
# Total: 12 processes × 500MB = 6GB

# Without process type separation:
10 × (web + worker) = 10GB — 4GB wasted on idle workers
```

### Process Formation

The set of process types and counts is called the **process formation**:

```
# Procfile (Heroku)        # Kubernetes
web: ./server              replicas: 10
worker: ./worker           ...
```

**Note:** Threads, goroutines, and event loops are fine inside a single process. But the unit of scaling is always the process — because processes can move across machines; threads cannot.

**Note:** A 12-factor app never daemonizes itself and never writes a PID file. Process lifecycle management is the platform's job.

---

## Factor 9 — Disposability

> *Maximize robustness with fast startup and graceful shutdown.*

Processes are started and stopped constantly — deployments, autoscaling, container restarts. This is not an incident; it's normal. Design for it.

### Fast Startup

A slow boot (60s) means:
- Rolling deploy of 10 instances takes 10 minutes
- Autoscaler is 1 minute late responding to traffic spikes

Target: a few seconds maximum.

### Graceful Shutdown (SIGTERM)

**Web process:**
1. Stop accepting new requests
2. Finish all in-flight requests
3. Exit cleanly

**Worker process:**
1. Return the current job back to the queue (don't discard it)
2. Exit cleanly

**Re-entrancy / idempotency requirement:** A job returned to the queue will be processed again. The processing code must be safe to run multiple times — either wrap in a transaction or make the operation idempotent (running it twice produces the same result as running it once).

### Crash-Only Design

SIGTERM is the polite case. Hardware failures, OOM kills — no warning. If your process survives crashes gracefully (crash-only design), the crash path **is** the shutdown path and no separate graceful shutdown code is needed.

---

## Factor 10 — Dev/Prod Parity

> *Keep development, staging, and production as similar as possible.*

The document identifies three gaps between development and production:

| Gap | Traditional | 12-Factor |
|-----|-------------|-----------|
| **Time** | Deploy every 1–2 weeks | Deploy every few hours |
| **Personnel** | Dev writes, Ops deploys, Ops debugs | Same person writes, deploys, and reads logs |
| **Tools** | SQLite on Mac, Postgres on Linux server | Same database in all environments |

### The Tools Gap — A Dangerous Example

ORM-based apps can swap databases transparently — which tempts developers to use SQLite locally and Postgres in production.

**Bug:** In Postgres, `LIKE` is case-sensitive; `ILIKE` is case-insensitive. In SQLite, `LIKE` is case-insensitive for ASCII by default. A search feature built locally with SQLite produces wrong results silently in production. No error. No warning. Just changed behavior.

**Rule:** Use the same backing service type and major version in local, staging, and production.

### The Managed Services Tension

In 2026, DynamoDB, Cloud Spanner, and other proprietary managed services have no exact local equivalent. LocalStack emulates many AWS services but isn't the same implementation. **Nuanced takeaway:** If the benefits of a managed service outweigh the dev/prod gap it creates, use it — but be deliberate about what behaviors you're not testing locally.

---

## Factor 11 — Logs

> *Treat logs as event streams.*

**Mental model shift:** A log is not a file. A log is a **stream of time-ordered events** that flows continuously as long as the app runs. The file is just one possible output format.

### The Rule

Your app writes all log events, unbuffered, to **stdout** (standard output). It does not manage log files, does not rotate logs, does not route logs. That is the platform's responsibility.

```
App → stdout → platform captures → routes to:
  - Terminal (local development)
  - Log aggregator with process metadata (Docker/Kubernetes)
  - Log drain service (Loki, Datadog, Splunk) in production
```

### Why Streams Over Files

| Property | File-based | Stream-based |
|----------|------------|--------------|
| Multi-instance aggregation | Manual, per-file | Platform merges all streams, ordered by time |
| Filtering | grep on files | Pipe to any tool |
| Routing | Hard-coded in app | Config-driven by platform |
| Code changes per environment | Yes | No |

**The composition advantage:** In local, you read logs in the terminal. In Docker, the runtime stamps each log with which container it came from. In production with 10 instances, the platform merges all streams into one unified, time-ordered log — no per-instance querying needed.

**Modern extension:** The 2011 document only covers logs. Today's observability stack adds metrics and traces. Treat all three as streams the platform collects and routes — the same principle.

---

## Factor 12 — Admin Processes

> *Run admin and management tasks as one-off processes.*

**One-off process:** A task that runs once, alongside the long-running app processes, then exits.

**Examples:**
- Database migrations
- One-time data fix scripts
- Manually triggered report generators
- REPL session to inspect live data

### The Rule

One-off processes must run in the same environment as the app's regular processes:
- Same codebase (same commit)
- Same config
- Same dependency isolation (same container image)

**Why:** The alternative is SSH-ing into a production server and running a raw SQL `UPDATE` or `DELETE`. No code review. No audit trail. No way to reproduce. Eventual drift between production state and the documented codebase.

The most common example in every codebase: **database migrations** — shipped with the same code, run as a one-off process before the server starts, then exit.

---

## Demo — Factors in Practice

The Tasker app running on a Linux machine, managed by a **Procfile** (Heroku's process formation format) and **Overmind** (a process manager):

```
# Procfile
web: ./server
worker: ./worker
```

**Boot sequence (factors 2, 3, 5, 8):**
1. Schema check (migration one-off)
2. Background job server starts
3. Server binds to assigned port (Factor 7)
4. All log events stream to stdout (Factor 11)

**Second terminal — admin one-off (Factor 12):**
```bash
overmind run web ./scripts/auto-archive  # same env, same code, exits when done
```

**Ctrl-C — graceful shutdown (Factor 9):**
1. Connection pool closes
2. Job server waits for all workers to finish current jobs
3. Process reports clean exit

---

## Summary

| Factor | One-liner | Key rule |
|--------|-----------|----------|
| **1 Codebase** | One repo, many deploys | Code is shared; config makes deploys different |
| **2 Dependencies** | Declare + isolate | Manifest file + lock file + isolation (virtualenv / Docker) |
| **3 Config** | Environment variables | Secrets in secret manager; config never in code |
| **4 Backing services** | Attached resources | URL + credentials = resource handle; code is identical for local vs managed |
| **5 Build/Release/Run** | Three strict stages | Releases are immutable; rollback is instant; no SSH hot-patching |
| **6 Processes** | Stateless, share nothing | State in backing services; sticky sessions = violation |
| **7 Port binding** | Self-contained | App brings its own HTTP server; exports via port (or declared interface) |
| **8 Concurrency** | Process model | Scale by running more processes; different types scale independently |
| **9 Disposability** | Fast start, clean stop | Finish in-flight work; return jobs to queue; idempotent operations |
| **10 Dev/prod parity** | Same everywhere | Same database type in local, staging, production |
| **11 Logs** | Streams, not files | Write unbuffered to stdout; platform routes the stream |
| **12 Admin processes** | One-off, same env | Same codebase + config; no SSH raw queries in production |
