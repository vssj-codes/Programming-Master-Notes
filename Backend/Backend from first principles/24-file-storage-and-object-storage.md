# Table-of-Contents

<!-- toc -->

- [File Storage and Object Storage](#file-storage-and-object-storage)
  * [The Naive Approach — Local File System](#the-naive-approach--local-file-system)
  * [Six Problems with Local File Storage](#six-problems-with-local-file-storage)
    + [1. Ephemeral Container Storage](#1-ephemeral-container-storage)
    + [2. Horizontal Scaling (Statelessness Violation)](#2-horizontal-scaling-statelessness-violation)
    + [3. Fixed Disk Size (Vertical Only)](#3-fixed-disk-size-vertical-only)
    + [4. Durability](#4-durability)
    + [5. Bandwidth Cost (Server as CDN)](#5-bandwidth-cost-server-as-cdn)
    + [6. No Transaction Between File System and Database](#6-no-transaction-between-file-system-and-database)
  * [Three Types of Storage](#three-types-of-storage)
    + [Block Storage](#block-storage)
    + [File Storage (POSIX)](#file-storage-posix)
    + [Object Storage](#object-storage)
  * [Object Structure](#object-structure)
  * [Folders Are an Illusion](#folders-are-an-illusion)
  * [Key Design Rules](#key-design-rules)
  * [How Object Storage Works Internally](#how-object-storage-works-internally)
    + [Data Plane](#data-plane)
    + [Metadata Plane](#metadata-plane)
  * [Conditional Writes (Recent Feature)](#conditional-writes-recent-feature)
  * [Upload Architectures](#upload-architectures)
    + [Architecture 1: Through Your Server](#architecture-1-through-your-server)
    + [Architecture 2: Pre-Signed URLs (Recommended)](#architecture-2-pre-signed-urls-recommended)
    + [Pre-Signed POST with Policy (Add Size + Type Enforcement)](#pre-signed-post-with-policy-add-size--type-enforcement)
  * [Two-Phase Upload Flow (The Production Pattern)](#two-phase-upload-flow-the-production-pattern)
  * [CORS Configuration](#cors-configuration)
  * [Summary](#summary)

<!-- tocstop -->

---

# File Storage and Object Storage

**Source:** Sriniously — Backend from First Principles (Video 24)
**Link:** [Watch](https://www.youtube.com/watch?v=estH64OkwxU)

---

## The Naive Approach — Local File System

When first implementing file uploads, the intuitive approach is:
1. Receive file via `multipart/form-data`
2. Save it to the server's local file system
3. Save the file path in the database

This approach breaks immediately in production for six distinct reasons.

---

## Six Problems with Local File Storage

### 1. Ephemeral Container Storage

Modern servers run inside Docker containers deployed with Kubernetes or PaaS platforms. Container file systems are **ephemeral** — every time a container restarts (crash, memory OOM kill, deployment), a brand new container is created from the image. The image does not contain user files. Everything stored in the container's local file system is wiped.

### 2. Horizontal Scaling (Statelessness Violation)

With three server instances behind a load balancer, a file uploaded to instance 1 only exists on instance 1's disk. When the browser later requests that file, the load balancer may route to instance 2 or 3 — which have no knowledge of the file → `404 Not Found`.

The more you scale out, the worse this gets. Probability of hitting the correct instance = 1/N. Storing files locally **breaks statelessness**, the most important property of a horizontally scalable server.

### 3. Fixed Disk Size (Vertical Only)

Adding disk capacity is a vertical operation — you resize the existing disk (causing downtime). You cannot add disk capacity horizontally. Meanwhile, user-uploaded files grow continuously and never shrink (deletes are far rarer than uploads).

### 4. Durability

A single disk is a single point of failure. If the disk fails, all files are permanently gone (a **durability** failure). Durability failures cannot be recovered from by waiting — the data is simply gone.

> **Availability** = whether data can be reached right now (recoverable by waiting).
> **Durability** = whether data still exists somewhere in the world (unrecoverable if lost).

The only solution to durability is **replication** — keeping multiple copies in different failure domains.

### 5. Bandwidth Cost (Server as CDN)

Serving files from your server means a worker (goroutine, event loop slot) sits idle streaming bytes slowly to a mobile connection for minutes. Your server is not a CDN — using it as one is expensive, slow, and wastes concurrency capacity for real request processing.

### 6. No Transaction Between File System and Database

File system and database are two independent systems — you cannot wrap both in a single transaction. Whichever operation happens first can succeed while the second fails:
- Write file first, then DB insert fails → orphaned file (costs money, inaccessible)
- DB insert first, then file write fails → database row points to a non-existent path → `500` or `404` for every request

---

## Three Types of Storage

### Block Storage

A raw array of fixed-size blocks (~512 bytes or 4KB each), addressable only by block number. No concept of files, names, or folders.

- **Examples:** AWS EBS (Elastic Block Store), physical SSD
- **Latency:** Microseconds
- **Use case:** Database systems (Postgres, MySQL sit on top of it)
- **Critical limitation:** A block device can be attached to exactly one machine at a time

### File Storage (POSIX)

An abstraction built on top of block storage. Provides hierarchical structure (directories, files, paths), permissions, and a rich interface: open, close, seek, overwrite specific bytes, rename (atomically), locking, append.

- **Examples:** ext4, APFS, NFS
- **Latency:** Microseconds (local), milliseconds (network)
- **Trade-off:** All those rich operations require coordination — locking, atomic renames, consistent partial writes — which does not scale across network boundaries and multiple continents

### Object Storage

Starts from a completely different question: **what is the absolute minimum interface to achieve unlimited scale?**

**The four operations:**
1. `PUT` — upload an object with a name (key)
2. `GET` — retrieve an object by key
3. `DELETE` — delete an object by key
4. `LIST` — list all keys (with optional prefix filter)

That's everything.

**What you give up:**
- No in-place modification — objects are immutable; to change one byte, download the whole object, modify, re-upload
- No `SEEK` or partial write
- No real directories (illusion only)
- No atomic rename (copy + delete, costs full object size)
- No file locking
- Latency: milliseconds (every operation is an HTTP request over a network)

**What you gain:**
- No coordination needed → horizontal scaling without limits
- Capacity is effectively infinite — no provisioning, no resizing
- 11 nines of durability (99.999999999%)
- Every object is addressable via plain HTTP — any device speaks HTTP
- Any of thousands of servers worldwide can serve any request

> **The universal distributed systems trade-off:** Every time you achieve infinite scale, you give up mutability (the ability to modify state in place). Object storage is the purest example of this trade-off.

---

## Object Structure

Each object has four components:

| Component | Description |
|-----------|-------------|
| **Key** | A string — the complete identity of the object |
| **Value** | An array of bytes (binary blob) |
| **System metadata** | Size, last modified time, content type, ETag |
| **User metadata** | Custom key-value pairs you attach (e.g., `uploaded-by: user_id`) |

Objects live inside a **bucket** — a named container. `bucket name + key` must be globally unique across the entire platform.

---

## Folders Are an Illusion

There are no real directories in object storage. What looks like a folder structure in the AWS console (`uploads/2026/cat.png`) is just one object whose key is the string `uploads/2026/cat.png`. The slashes have no special operational meaning — they're just characters in a string.

**How the illusion works:** When you call `list` with a prefix `uploads/` and delimiter `/`, the object storage scans all keys starting with `uploads/`, chops each one at the first `/`, and groups them as "common prefixes." This is a string group-by algorithm, not a real folder structure.

**Consequences:**
- Moving a "folder" = copying all objects to new keys + deleting old ones → costs full object size × N objects
- Empty folders are impossible (a folder has no identity) — faked by a zero-byte object ending in `/`

---

## Key Design Rules

**Rule 1: Never use the user-provided filename as the key.**

Three reasons:
- **Collisions:** Two users uploading `resume.pdf` → second upload silently overwrites the first (PUT semantics)
- **Path traversal:** Special characters (`.`, `..`) can escape your intended prefix
- **Injection:** Null bytes, emoji, right-to-left override characters → bugs in your backend

**Solution:** Generate the key yourself (UUID or random string). Store the original filename in your database or in user metadata on the object.

**Rule 2: Put high-entropy values at the start of the key, not the end.**

Object storage partitions its index by key prefix. Keys like `2026/01/15/<uuid>` mean all uploads in the next hour share the same prefix → all hit the same index partition → **hot partition** → `503 Slowdown` errors under load.

Keys like `<uuid>/<date>/<filename>` or `<tenant-id>/<user-id>/<uuid>` spread load across partitions.

**Rule 3: Make the key carry ownership.**

Keys like `<tenant-id>/<user-id>/<object-id>` allow authorization to be a simple string comparison (does the key start with this user's prefix?), skipping a database roundtrip per request.

---

## How Object Storage Works Internally

When a PUT request arrives, the system splits into two planes:

### Data Plane

Stores the actual bytes durably. Uses **erasure coding** instead of simple 3× replication:

- Split object into 14 data shards
- Compute 6 parity shards using Reed-Solomon algorithm
- Scatter all 20 shards across 20 drives in different racks, power domains, buildings
- **Any 14 of the 20 shards** are sufficient to reconstruct the original object

**Cost comparison:**
- Simple 3× replication: 3× storage overhead, tolerates 2 failures
- Erasure coding (14+6): 1.43× overhead, tolerates 6 simultaneous failures

**AWS S3's 11 nines guarantee (99.999999999% durability):** Statistically, if you store 10 million objects, you'd expect to lose one once every 10,000 years.

> **Durability ≠ Backup.** The 11 nines protect against drives dying. They do not protect against you running `DELETE` on the wrong prefix, or your app overwriting objects due to a bug. Enable **versioning** and replicate to a second account in a different AWS account for true protection.

### Metadata Plane

Maintains a distributed sorted index mapping `bucket + key → shard locations + size + ETag + content type`. Conceptually a distributed B-tree.

- Handles enormous query volume at very low latency
- Must be **strongly consistent**

**Strongly consistent** means: after a successful PUT, any subsequent GET from any server anywhere will return the object — even a millisecond later. (S3 achieved this in December 2020; before that it was eventually consistent, and you'd get `404` immediately after upload.)

> **Never use `list` in the request path.** Listing is a range scan over a distributed sorted index — much more expensive than a key lookup. Your database should be the index of what files exist. The bucket is only for the actual bytes.

---

## Conditional Writes (Recent Feature)

Before 2024, a PUT always overwrote any existing object at the same key — silently, with no way to detect or prevent it.

**Two new request headers:**

| Header | Value | Behavior |
|--------|-------|---------|
| `if-none-match` | `*` | Only write if this key does NOT already exist → `412 Precondition Failed` if it does |
| `if-match` | `<etag>` | Only write if the object's current ETag matches → `412` if someone else modified it since you read it |

**`if-none-match: *`** = a distributed lock primitive. Guarantees exactly one writer wins when two clients race to create the same key.

**`if-match`** = compare-and-swap. Client A reads object (gets ETag), modifies locally, uploads with `if-match: <original-etag>`. If Client B updated the object in between, Client A's upload gets `412` instead of silently overwriting B's data.

This is why object storage is increasingly used as the foundation for distributed systems, write-ahead logs, and databases — it now provides the coordination primitive needed to guarantee exactly-one-writer semantics.

> **Check compatibility:** Not all "S3-compatible" providers support conditional writes (e.g., Backblaze B2 does not). Verify before depending on this feature.

---

## Upload Architectures

### Architecture 1: Through Your Server

Browser → POST multipart → Your server → forward to S3

**Advantage:** Server sees every byte; can validate, scan, hash, reject before storage.

**The buffering problem:** Most web frameworks read the entire file into memory before handing it to your handler. A 100MB upload × 20 concurrent users = 2GB RAM consumed instantly → OOM kill → container restart → files lost.

**Solution if using this pattern: Stream instead of buffer.** Read from the request body in chunks and pipe those chunks directly to the S3 SDK upload as they arrive. Memory usage stays flat at ~one buffer regardless of file size.

```go
// Go example: stream directly
part, _ := reader.NextPart()
uploader.Upload(&s3.UploadInput{
    Body: part,  // reader, not []byte
})
```

Always wrap the body in a size limit before reading anything — cut off oversized requests at the boundary before they consume memory.

**Remaining limits:** Load balancer idle timeout (~60s default), reverse proxy body limits (Nginx `413 Request Entity Too Large`), serverless hard payload caps (a few MB, non-configurable).

**Verdict:** Works for small files (profile pictures, CSVs under a few MB). Not recommended for anything larger.

---

### Architecture 2: Pre-Signed URLs (Recommended)

Browser uploads **directly to S3** — bytes never touch your server.

**The problem this solves:** Bucket must be private (exposing credentials to the browser is a major security risk). Users need to upload without having credentials.

**How pre-signed URLs work:**

A pre-signed URL is a normal URL with query parameters: algorithm, access key ID, timestamp, expiry (in seconds), covered headers, and a **signature** (HMAC of the method + bucket + key + expiry + headers, hashed with your secret key).

```
https://bucket.s3.amazonaws.com/key
  ?X-Amz-Algorithm=AWS4-HMAC-SHA256
  &X-Amz-Credential=<access-key>/...
  &X-Amz-Date=<timestamp>
  &X-Amz-Expires=300
  &X-Amz-SignedHeaders=content-type
  &X-Amz-Signature=<hmac>
```

When S3 receives the upload, it recomputes the HMAC using its own copy of the secret and compares. If the browser changes even one character of the key, method, or tries to use it after expiry → `403 Forbidden`.

**The credential never leaves your server.** The browser gets upload capability for exactly one object for exactly N seconds.

---

### Pre-Signed POST with Policy (Add Size + Type Enforcement)

A plain pre-signed PUT URL has no enforced size limit — a user can upload a 5GB file to a URL signed for a profile picture.

**Fix:** Sign a JSON policy document instead of just a URL. The policy includes conditions:

```json
{
  "conditions": [
    ["content-length-range", 1, 5242880],      // 1 byte to 5MB
    ["starts-with", "$Content-Type", "image/"], // must be an image
    ["starts-with", "$key", "tenants/user123/"] // must be in user's prefix
  ]
}
```

S3 enforces these conditions itself — your server never sees the upload. Violations return `400 Entity Too Large` or `403 Forbidden`.

---

## Two-Phase Upload Flow (The Production Pattern)

**The problem:** If the browser uploads directly to S3, how does your server know it happened? How does the database get updated?

**Solution: Two-phase flow**

**Phase 1 — Intent Registration (before any bytes move):**
1. Browser calls your backend: "I want to upload a profile picture"
2. Backend authenticates/authorizes the user
3. Backend **generates the object key** (random UUID — not user-provided filename)
4. Backend writes a **database row** with status `pending` + the generated key + expected constraints
5. Backend returns the pre-signed POST URL + upload ID to the browser

**Phase 2 — Confirmation (after upload):**
1. Browser uploads directly to S3 using the pre-signed URL
2. Browser calls your backend completion endpoint with the upload ID (not the key)
3. Backend performs `HeadObject` on S3 — a metadata-only call that verifies:
   - Object exists
   - Size is within expected range
   - Content type is expected
4. If all checks pass, backend updates database row status from `pending` → `completed`

**Cleanup for abandoned uploads:**
- Some uploads will be abandoned (network drops, tab closed, slow connections)
- Set an S3 lifecycle rule to auto-delete objects under the `pending/` prefix after 24 hours
- A background job scans pending database rows older than 24 hours and marks them `expired`
- This prevents the database and bucket from drifting out of sync (database is source of truth for what exists; bucket is source of truth for the actual bytes)

---

## CORS Configuration

When a browser uploads directly to S3 (a different origin than your backend), it's a cross-origin request — CORS applies.

Your bucket must have a CORS policy that explicitly:
- Allows your frontend domain
- Allows `PUT` and `POST` methods
- Exposes the `ETag` header (needed for multipart uploads)

If an upload works in `curl` or Postman but fails in the browser → check your bucket's CORS configuration.

---

## Summary

| Concern | Key point |
|---------|----------|
| **Local file system** | Fails in production: ephemeral, not stateless, fixed size, no durability, bandwidth cost, no transactions |
| **Block storage** | Lowest latency (µs); for databases; cannot be shared between machines |
| **File storage** | Rich POSIX interface; coordination overhead doesn't scale globally |
| **Object storage** | Minimal interface (put/get/delete/list); immutable objects; infinite horizontal scale |
| **No real folders** | Keys with `/` are just strings; folders are a group-by illusion |
| **Key design** | Never use user filename; high entropy at front; carry ownership in key structure |
| **Erasure coding** | 14+6 shards; 1.43× storage overhead vs 3× replication; tolerates 6 failures; 11 nines durability |
| **Strong consistency** | S3 strongly consistent since Dec 2020; GET after PUT always succeeds |
| **Conditional writes** | `if-none-match: *` = distributed lock; `if-match: <etag>` = compare-and-swap |
| **Upload via server** | Works for small files; must stream (never buffer); size-limit the body |
| **Pre-signed URL** | Browser uploads directly to S3; credentials never exposed; time-limited; recommended |
| **Policy signing** | Enforce size limits and content type at the bucket level, not your server |
| **Two-phase flow** | Server pre-creates DB record → browser uploads → server confirms via HeadObject |
| **CORS** | Required when browser uploads directly to bucket; allow PUT/POST + expose ETag |
