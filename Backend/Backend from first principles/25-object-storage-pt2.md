# Table-of-Contents

<!-- toc -->

- [Large File Uploads and Downloads](#large-file-uploads-and-downloads)
  * [Why a Single PUT Fails for Large Files](#why-a-single-put-fails-for-large-files)
    + [Reason 1: Hard Limit](#reason-1-hard-limit)
    + [Reason 2: All-or-Nothing — No Resume](#reason-2-all-or-nothing--no-resume)
    + [Reason 3: Single TCP Throughput Ceiling](#reason-3-single-tcp-throughput-ceiling)
    + [Reason 4: Poor User Experience](#reason-4-poor-user-experience)
  * [Multipart Upload — The Protocol](#multipart-upload--the-protocol)
    + [Call 1: Create Multipart Upload](#call-1-create-multipart-upload)
    + [Call 2: Upload Part (Once Per Chunk)](#call-2-upload-part-once-per-chunk)
    + [Call 3: Complete Multipart Upload](#call-3-complete-multipart-upload)
    + [Call 4: Abort Multipart Upload](#call-4-abort-multipart-upload)
  * [Part Size Strategy](#part-size-strategy)
  * [The ETag Gotcha on Multipart Uploads](#the-etag-gotcha-on-multipart-uploads)
  * [Abandoned Uploads Cost Money](#abandoned-uploads-cost-money)
  * [Full Production Architecture: Pre-Signed URLs + Multipart](#full-production-architecture-pre-signed-urls--multipart)
    + [Resumable Uploads — Automatic](#resumable-uploads--automatic)
  * [Download Patterns](#download-patterns)
    + [Pattern 1: Public Content via CDN (Cheapest + Fastest)](#pattern-1-public-content-via-cdn-cheapest--fastest)
    + [Pattern 2: Pre-Signed GET](#pattern-2-pre-signed-get)
  * [Range Requests](#range-requests)
  * [Adaptive Bitrate Streaming (HLS / DASH)](#adaptive-bitrate-streaming-hls--dash)
  * [Pricing Model](#pricing-model)
  * [Summary](#summary)

<!-- tocstop -->

---

# Large File Uploads and Downloads

**Source:** Sriniously — Backend from First Principles (Video 25)
**Link:** [Watch](https://www.youtube.com/watch?v=estH64OkwxU)

> Continuation of Video 24. Covers why single PUT breaks for large files, the multipart upload protocol, download architectures, range requests, adaptive streaming, and pricing.

---

## Why a Single PUT Fails for Large Files

### Reason 1: Hard Limit

A single PUT request cannot exceed **5GB** — enforced by the object storage API itself, not by your backend or any specification. Any product accepting large video, CAD files, database dumps, or drone footage is architecturally required to use something else.

### Reason 2: All-or-Nothing — No Resume

A single PUT has no resume capability. A network glitch at 4.8GB of a 5GB upload cancels the entire operation — the next attempt starts from zero. On typical mobile networks (10–50 Mbps), a 5GB file takes 20–30 minutes to upload. A single blink of Wi-Fi mid-transfer means starting over.

### Reason 3: Single TCP Throughput Ceiling

A single TCP connection cannot saturate a network link. Throughput is bounded roughly by:

```
throughput ≈ window_size / round_trip_time
```

And it is extremely sensitive to packet loss — a single lost packet collapses the congestion window, which must climb back up from scratch. A 500 Mbps connection can stall at 30 Mbps on a single stream.

**Fix at the network level:** multiple parallel TCP connections, each with its own independent congestion window. This is exactly what download managers (IDM, Free Download Manager) exploit — they open 8 connections to the same object and download in parallel, boosting throughput without affecting the physical link.

### Reason 4: Poor User Experience

A single PUT gives imprecise progress, no ability to pause, and no ability to survive a disconnection or tab close.

---

## Multipart Upload — The Protocol

Three API calls constitute the entire protocol.

### Call 1: Create Multipart Upload

Send: bucket, key, content type.
Receive: **upload ID** — a staging area identifier.

The staging area is analogous to git's index — parts accumulate here before the final "commit." At this point the object does not exist at the key. A GET on that key returns `404` until completion.

### Call 2: Upload Part (Once Per Chunk)

Send: upload ID, part number, bytes.

Key properties:
- Each part is a **completely independent HTTP request** — fully parallelizable
- Parts can arrive in **any order** (part 7 before part 2 is fine)
- Each failed part can be **retried individually** without affecting others
- Each successful response returns an **ETag** — save every one, they're needed for completion

### Call 3: Complete Multipart Upload

Send: upload ID + ordered list of `{ part_number, etag }` pairs.

The service records the structure of the object (which parts, in which order) in the metadata plane. It does **not** physically concatenate gigabytes of data — completion time is effectively constant regardless of object size or part count.

### Call 4: Abort Multipart Upload

Cancels and deletes all parts in the staging area for a given upload ID.

---

## Part Size Strategy

**Hard cap:** 10,000 parts maximum.

- Default 5MB parts × 10,000 = 50GB max file — any 60GB file fails at part 10,001 after 50GB transferred (expensive failure)
- Default 16MB parts × 10,000 = 160GB max
- Default 32MB parts × 10,000 = 320GB max

**Correct formula — dynamic part size:**

```
part_size = max(floor_size, ceil(file_size / 10_000))
```

Use a floor of 16–32MB. This handles files up to 160–320GB without touching the formula, and avoids creating thousands of tiny parts for small files.

**Why not use huge parts (500MB+)?** Parts are your retry granularity. If a 500MB part fails on a mobile connection, you retry 500MB. With 16–32MB parts, retries are cheap.

**Sweet spot: 16MB–64MB per part.**

---

## The ETag Gotcha on Multipart Uploads

For a regular single PUT, the ETag = MD5 hash of the object's bytes. Many implementations write integrity checks comparing a locally computed MD5 to the returned ETag.

**This breaks silently with multipart uploads.**

For multipart objects, the ETag format is:

```
<md5_of_concatenated_part_md5s>-<number_of_parts>
```

Example: `d41d8cd98f00b204e9800998ecf8427e-42`

**Correct integrity approaches:**

| Approach | How |
|----------|-----|
| **Modern SDK checksums** | AWS S3 SDK computes CRC32C or SHA-256 per part and lets you request a full-object checksum; the service verifies it |
| **User metadata hash** | Compute SHA-256 of the full file on the client, attach as user metadata on upload, verify in a background job after completion |

The user metadata approach also gives deduplication for free.

---

## Abandoned Uploads Cost Money

If a browser uploads 300 parts (each 32MB = ~10GB of data) to the staging area and then closes the tab before calling complete or abort, those parts remain in the staging area **indefinitely**.

- They do not appear in bucket listings (the object does not exist yet)
- They do not appear in the AWS console's object count
- **You are billed for them every month**

**Verification:**
```bash
aws s3api list-multipart-uploads --bucket <bucket-name>
```

**Fix — lifecycle rule (set on every bucket before writing any code):**

> Abort incomplete multipart uploads after **7 days**.

7 days is enough for any legitimate slow upload to complete, while permanently closing the cost leak.

---

## Full Production Architecture: Pre-Signed URLs + Multipart

```
Browser ──POST /uploads/init──► Backend
         {filename, size, type}
                                 ├─ auth/authz check
                                 ├─ compute dynamic part_size
                                 ├─ call CreateMultipartUpload → uploadId
                                 ├─ insert DB row (status: pending)
                                 └─ pre-sign URL for each part (or first batch)

Backend ──► Browser
         {uploadId, partSize, presigned_urls[]}

Browser ──N parallel PUT requests──► S3 (direct, never via backend)
         (N = 4–6 concurrent)
         Collects ETags from each response

Browser ──POST /uploads/complete──► Backend
         {uploadId, parts: [{partNumber, etag}]}
                                 ├─ call CompleteMultipartUpload
                                 ├─ HeadObject verification
                                 │  (size, content-type, part count)
                                 ├─ update DB row (status: completed)
                                 └─ emit file.uploaded event to queue
```

**Implementation notes:**

| Point | Detail |
|-------|--------|
| **Batch URL pre-signing** | Pre-sign in batches of ~100, not all 10,000 upfront. An all-at-once response becomes megabytes of JSON |
| **Concurrency** | 4–6 parallel part uploads is the sweet spot; beyond that, connections compete for bandwidth |
| **Browser memory** | `File.slice()` returns a lazy Blob — memory usage ≈ `part_size × concurrency`, not total file size |
| **CORS** | Must expose the `ETag` response header in the bucket's CORS config, or JavaScript cannot read part ETags |

### Resumable Uploads — Automatic

On page refresh, the browser asks the backend: "do I have an in-progress upload for this file?"

Backend:
1. Looks up the `pending` DB row
2. Calls `ListParts` with the upload ID → finds which parts the bucket already has
3. Returns fresh pre-signed URLs for only the missing parts

The client resumes from part 341 (for example) instead of starting over.

---

## Download Patterns

**Rule 0: Never proxy downloads through your server.**

Relaying downloads via your backend:
- Occupies a worker (goroutine, event loop slot) for the entire download duration
- You pay egress bandwidth twice (bucket → server + server → user)
- Slower than CDN (server is in one or two regions; CDN is global)

### Pattern 1: Public Content via CDN (Cheapest + Fastest)

For objectively public assets (JS/CSS bundles, product images, static site content):

- Keep the bucket **private**
- Put a CDN in front; only the CDN reads the bucket
- Use **content-addressed keys** (key = hash of file contents)
- Set very long `Cache-Control` headers
- URL changes automatically when content changes → no cache invalidation needed

### Pattern 2: Pre-Signed GET

For private files the user must be authorized to access:

1. Browser requests a file from your backend
2. Backend checks authorization
3. Backend returns a pre-signed GET URL (valid 5–15 minutes)
4. Browser fetches directly from S3/CDN using that URL

**The CDN caching problem:** Every pre-signed GET URL contains a unique timestamp + signature → different URL per request → CDN treats each as a different resource → **cache hit rate = 0%**.

**Fixes:**

| Fix | How | Trade-off |
|-----|-----|-----------|
| **Round-up expiry** | Sign with `valid until top of next hour` instead of `valid for 15 minutes from now` | All users in the same hour get the same URL; slightly coarser expiry |
| **CDN signed URLs / cookies** | Authorization happens at the CDN edge, not your backend; CDN controls caching entirely | More complex setup; correct approach at scale |

**Signed cookies** are especially useful for video: a single video session fetches hundreds of segments. Signing each one via your backend is impractical; a CDN cookie covers the entire session.

---

## Range Requests

HTTP supports requesting a byte range of any resource:

```
GET /object HTTP/1.1
Range: bytes=1000000-3000000
```

Responses:
- `206 Partial Content` + `Content-Range: bytes 1000000-3000000/50000000`
- `416 Range Not Satisfiable` — if the range is invalid

Object storage natively supports this HTTP semantic. Combining client-side range requests with S3's native range-read capability unlocks:

| Use case | Mechanism |
|----------|-----------|
| **Video seeking** | Browser calculates byte offset for timestamp, issues range request from there — no need to download skipped portion |
| **Resumable downloads** | Download manager stores last received byte; on reconnect, re-requests from that byte |
| **Parallel downloads** | Open 6 connections for the same object, each covering a different range — independent congestion windows |

---

## Adaptive Bitrate Streaming (HLS / DASH)

Object storage serves exactly one version of a file — a limitation for video, where connection quality varies per user.

**The approach:**

1. **Transcode** the original video into multiple quality levels (360p, 720p, 1080p, 4K)
2. **Segment** each quality version into small chunks (a few seconds each)
3. **Write a manifest file** (`.m3u8` for HLS, `.mpd` for DASH) listing all segments and quality mappings

The player:
- Fetches the manifest first
- Fetches segments one at a time
- Measures how long each segment fetch took → selects higher or lower quality for the next segment
- **Adaptive bitrate** = the player continuously adjusts quality based on measured throughput

From the object storage perspective: segments and the manifest file are all ordinary objects. Object storage + CDN covers the serving layer completely. The hard engineering is the **transcoding pipeline** — the background workers triggered on upload that produce all quality levels and the manifest.

> If video is a minor feature of your product, use a hosted service (Mux, Bunny, Cloudinary). Build the pipeline yourself only if video is the core product and the cost has to be optimized.

---

## Pricing Model

| Cost type | Measured by | Notes |
|-----------|-------------|-------|
| **Storage** | GB per month | Includes parts in incomplete multipart uploads |
| **Operations** | Per request | Writes (PUT/DELETE) are ~10× more expensive than reads (GET) |
| **Egress** | Data leaving the cloud provider | Usually the dominant cost at scale; Cloudflare R2 charges $0 egress — significant for high-read workloads |

---

## Summary

| Concept | Key point |
|---------|----------|
| **Single PUT limit** | Hard 5GB cap; no resume; single-TCP throughput ceiling; poor UX |
| **Multipart upload** | CreateMultipartUpload → N parallel UploadPart → CompleteMultipartUpload |
| **Staging area** | Object does not exist at key until Complete — GET returns 404 during upload |
| **Part ordering** | Parts arrive in any order; each retried independently |
| **Part size** | Dynamic: `max(floor, ceil(size / 10_000))`; 16–32MB floor is safe to ~160–320GB |
| **ETag gotcha** | Multipart ETags are not MD5 of content — use SDK checksums or user-metadata SHA-256 |
| **Abandoned uploads** | Parts accumulate silently; add abort-incomplete lifecycle rule (7 days) to every bucket |
| **Full architecture** | Browser → backend init → backend pre-signs per-part URLs → browser uploads directly → browser calls complete → backend verifies via HeadObject |
| **Resumable uploads** | ListParts reveals which parts arrived; pre-sign URLs only for missing parts |
| **Never proxy downloads** | Ties up workers and doubles bandwidth cost; always serve from bucket/CDN directly |
| **Public CDN pattern** | Private bucket + CDN + content-addressed keys + long Cache-Control — cheapest and fastest |
| **Pre-signed GET** | Authorization at backend; CDN caching problem fixed by round-up expiry or CDN signed cookies |
| **Range requests** | HTTP 206; enables video seeking, resumable downloads, parallel downloads |
| **Adaptive streaming** | Transcode → segment → manifest; player measures throughput and switches quality |
| **Egress cost** | Dominant at scale; Cloudflare R2 offers free egress |
