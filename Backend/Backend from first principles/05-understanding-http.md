# Table-of-Contents

<!-- toc -->

- [Understanding HTTP for Backend Engineers](#understanding-http-for-backend-engineers)
  * [The Two Core Ideas of HTTP](#the-two-core-ideas-of-http)
    + [1. Statelessness](#1-statelessness)
    + [2. Client-Server Model](#2-client-server-model)
  * [HTTP and TCP](#http-and-tcp)
  * [HTTP Versions](#http-versions)
  * [HTTP Message Structure](#http-message-structure)
    + [Request Message](#request-message)
    + [Response Message](#response-message)
  * [HTTP Headers](#http-headers)
    + [Request Headers](#request-headers)
    + [General Headers](#general-headers)
    + [Representation Headers](#representation-headers)
    + [Security Headers](#security-headers)
    + [Two Key Ideas Behind Headers](#two-key-ideas-behind-headers)
  * [HTTP Methods](#http-methods)
    + [Idempotent vs Non-Idempotent](#idempotent-vs-non-idempotent)
  * [CORS (Cross-Origin Resource Sharing)](#cors-cross-origin-resource-sharing)
    + [Simple Request Flow](#simple-request-flow)
    + [Pre-flight Request Flow](#pre-flight-request-flow)
  * [HTTP Response Codes](#http-response-codes)
    + [1xx — Informational](#1xx--informational)
    + [2xx — Success](#2xx--success)
    + [3xx — Redirection](#3xx--redirection)
    + [4xx — Client Errors](#4xx--client-errors)
    + [5xx — Server Errors](#5xx--server-errors)
  * [HTTP Caching](#http-caching)
    + [Key Headers](#key-headers)
    + [Flow](#flow)
  * [Content Negotiation](#content-negotiation)
    + [Types](#types)
    + [HTTP Compression](#http-compression)
  * [Persistent Connections (Keep-Alive)](#persistent-connections-keep-alive)
  * [Handling Large Requests and Responses](#handling-large-requests-and-responses)
    + [Large Requests — Multipart](#large-requests--multipart)
    + [Large Responses — Chunked / Server-Sent Events](#large-responses--chunked--server-sent-events)
  * [SSL / TLS / HTTPS](#ssl--tls--https)
  * [Summary — What You Need to Internalize](#summary--what-you-need-to-internalize)

<!-- tocstop -->

---

# Understanding HTTP for Backend Engineers

**Source:** Sriniously — Backend from First Principles (Video 5)
**Link:** [Watch](https://www.youtube.com/watch?v=a3C1DMswClQ)

---

## The Two Core Ideas of HTTP

### 1. Statelessness

HTTP has **no memory of past interactions**. Every request:
- Is self-contained — carries all the information needed (headers, URL, method, auth tokens)
- Is treated by the server as new and unrelated after responding

**Benefits:**
- **Simplicity** — server does not store session info → less complexity
- **Scalability** — requests can be distributed across multiple servers; no single server needs to track sessions; server crash does not affect client state

> Because HTTP is stateless, developers layer on state management via cookies, sessions, or tokens (e.g. user logins, shopping carts).

### 2. Client-Server Model

- **Client** (browser, app) — initiates communication, provides the URL, headers, and all needed info
- **Server** — hosts resources/APIs, waits for requests, processes and responds

> Communication is **always initiated by the client**.

---

## HTTP and TCP

HTTP does not mandate a connection-based transport — only that it is *reliable*. In practice HTTP uses **TCP** (more reliable than UDP).

OSI model reference:
- Backend engineers mostly work at **Layer 7 (Application Layer)**
- TCP handshake, TLS — these are network-layer concerns (good to know, not to deep-dive)

---

## HTTP Versions

| Version | Key Change |
|---------|-----------|
| HTTP/1.0 | New TCP connection per request — slow |
| HTTP/1.1 | **Persistent connections** — reuse one TCP connection for multiple requests; chunked transfer encoding; better caching |
| HTTP/2.0 | **Multiplexing** — multiple requests/responses over one connection; binary framing; header compression (HPACK); server push |
| HTTP/3.0 | Built on **QUIC** (UDP-based); faster connection setup; lower latency; better packet loss handling; multiplexing without head-of-line blocking |

---

## HTTP Message Structure

### Request Message

```
METHOD /resource-url HTTP/1.1
Host: example.com
Header-Key: Header-Value
...
<blank line>
Request Body (optional)
```

### Response Message

```
HTTP/1.1 STATUS_CODE STATUS_TEXT
Header-Key: Header-Value
...
<blank line>
Response Body
```

---

## HTTP Headers

Headers are **key-value pairs** of metadata sent with requests and responses.

> Analogy: like the address label on a parcel — metadata written on the outside so intermediaries can route/handle it without opening the package.

### Request Headers
Sent by the client to describe the request:
- `User-Agent` — identifies the client (browser, Postman, mobile app)
- `Authorization` — credentials (e.g. Bearer token)
- `Accept` — preferred response format (JSON, HTML, XML)

### General Headers
Used in both requests and responses — metadata about the message itself:
- `Date`
- `Cache-Control` (no-cache, max-age)
- `Connection` (keep-alive / close)

### Representation Headers
Describe the body of the message:
- `Content-Type` — media type (application/json, text/html)
- `Content-Length` — size in bytes
- `Content-Encoding` — compression format (gzip, deflate)
- `ETag` — unique identifier for the resource (used in caching)

### Security Headers
Control browser behavior to harden security:
- `Strict-Transport-Security` (HSTS) — force HTTPS, prevent protocol downgrade
- `Content-Security-Policy` (CSP) — restrict sources for JS, CSS, images → prevents XSS
- `X-Frame-Options` — prevent iframe embedding → prevents clickjacking
- `X-Content-Type-Options` — prevent MIME-type sniffing
- `Set-Cookie` with `HttpOnly` + `Secure` flags — protect cookies

### Two Key Ideas Behind Headers

**Extensibility** — headers can be added/customized without changing the protocol:
- Custom headers (`X-Custom-Header`)
- Content negotiation (`Accept`, `Accept-Language`, `Accept-Encoding`)

**Remote control** — headers let clients instruct the server:
- Preferred format → `Accept`
- Cache behavior → `Cache-Control`, `Expires`
- Authentication → `Authorization`

---

## HTTP Methods

Methods define the **intent** of a request.

| Method | Purpose | Has Body |
|--------|---------|---------|
| `GET` | Fetch data — must not modify server state | No |
| `POST` | Create a new resource | Yes |
| `PATCH` | Partial update of a resource | Yes |
| `PUT` | Complete replacement of a resource | Yes |
| `DELETE` | Delete a resource | No |
| `OPTIONS` | Query server capabilities (used in CORS pre-flight) | No |

> Prefer `PATCH` over `PUT` unless you specifically need full replacement.

### Idempotent vs Non-Idempotent

**Idempotent** — calling multiple times produces the same result:
- `GET` — fetching the same data N times returns the same result
- `PUT` — replacing a resource N times ends in the same state
- `DELETE` — resource can only be deleted once; subsequent calls have the same outcome

**Non-idempotent:**
- `POST` — each call creates a new resource → different result each time

---

## CORS (Cross-Origin Resource Sharing)

Browsers enforce the **Same-Origin Policy** — web pages can only request resources from their own origin. CORS is the mechanism that lets servers opt in to cross-origin requests.

A cross-origin request = different **domain**, **port**, or **scheme** between client and server.

### Simple Request Flow

Conditions: method is `GET`, `POST`, or `HEAD` + no non-simple headers.

1. Browser adds `Origin` header to request
2. Server checks `Origin` against its CORS policy
3. If allowed, server responds with `Access-Control-Allow-Origin: <client-origin>` (or `*`)
4. Browser sees the header → lets response through
5. If header absent → browser **blocks** response (CORS error in console)

### Pre-flight Request Flow

Triggered when **any** of these are true (and the request is cross-origin):
- Method is `PUT`, `DELETE`, or other non-simple method
- Request includes non-simple headers (e.g. `Authorization`, custom headers)
- `Content-Type` is not `application/x-www-form-urlencoded`, `multipart/form-data`, or `text/plain` — so **JSON requests always trigger pre-flight**

**Pre-flight request (`OPTIONS`):**
```
OPTIONS /resource HTTP/1.1
Host: api.example.com
Origin: https://example.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: Authorization, Content-Type
```

**Pre-flight response (if CORS supported):**
```
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://example.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Authorization, Content-Type
Access-Control-Max-Age: 86400
```

- `Access-Control-Max-Age` — cache pre-flight result for N seconds (avoids repeated pre-flights)
- After successful pre-flight, browser sends the **original request**

---

## HTTP Response Codes

Three-digit numbers that communicate the result of a request without inspecting the body.

### 1xx — Informational
- `100 Continue` — server received headers, client can send body (used in large uploads)
- `101 Switching Protocols` — upgrading from HTTP to WebSocket

### 2xx — Success
| Code | Meaning | When to use |
|------|---------|------------|
| `200 OK` | Request successful | GET, general success |
| `201 Created` | Resource created | POST |
| `204 No Content` | Success, no body | DELETE, OPTIONS pre-flight response |

### 3xx — Redirection
| Code | Meaning | When to use |
|------|---------|------------|
| `301 Moved Permanently` | Resource permanently at new URL | Route renamed, backwards compat |
| `302 Found` | Temporary redirect | Campaign redirects |
| `304 Not Modified` | Resource unchanged, use cache | Conditional GET with ETags |

### 4xx — Client Errors
| Code | Meaning | When to use |
|------|---------|------------|
| `400 Bad Request` | Invalid/malformed data | Wrong type, missing fields |
| `401 Unauthorized` | Not authenticated | Missing/expired token |
| `403 Forbidden` | Authenticated but no permission | Accessing another user's resource |
| `404 Not Found` | Resource doesn't exist | Wrong URL, deleted resource |
| `405 Method Not Allowed` | Wrong HTTP method for route | PUT to a GET-only route |
| `409 Conflict` | State conflict | Duplicate folder/username |
| `429 Too Many Requests` | Rate limit exceeded | Client sending too many requests |

### 5xx — Server Errors
| Code | Meaning | When to use |
|------|---------|------------|
| `500 Internal Server Error` | Unexpected server failure | Unhandled exception |
| `501 Not Implemented` | Feature not yet supported | Planned but not built |
| `502 Bad Gateway` | Proxy got invalid upstream response | Nginx/load balancer issue |
| `503 Service Unavailable` | Server down / maintenance | High traffic, deployment |
| `504 Gateway Timeout` | Upstream didn't respond in time | Nginx timeout to app server |

---

## HTTP Caching

Stores copies of responses to avoid re-downloading unchanged data → reduces bandwidth and server load.

### Key Headers

**Server → Client (response):**
- `Cache-Control: max-age=10` — cache valid for 10 seconds
- `ETag: "abc123"` — hash/fingerprint of the response body
- `Last-Modified: <date>` — when the resource was last changed

**Client → Server (conditional request):**
- `If-None-Match: "abc123"` — only send new data if ETag differs
- `If-Modified-Since: <date>` — only send new data if modified after this date

### Flow

1. **First request** — server returns `200` + body + `ETag` + `Cache-Control` + `Last-Modified`
2. **Subsequent request (within max-age)** — client sends `If-None-Match` / `If-Modified-Since`
3. **Resource unchanged** → server returns `304 Not Modified` (no body) → client uses cached version
4. **Resource changed** → server returns `200` + new body + new `ETag`

> Modern alternative: **React Query** gives the client full control over caching strategy (preferred over manual HTTP caching for most frontend use cases).

---

## Content Negotiation

Mechanism for client and server to **agree on the best format** to exchange data.

### Types

| Type | Header | Example values |
|------|--------|---------------|
| Media type | `Accept` | `application/json`, `application/xml`, `text/html` |
| Language | `Accept-Language` | `en`, `es`, `fr` |
| Encoding | `Accept-Encoding` | `gzip`, `deflate`, `br` |

Server reads these and responds in the client's preferred format when available.

### HTTP Compression

When a large response is involved, the server compresses the body:
- Client sends `Accept-Encoding: gzip, deflate, br`
- Server compresses and responds with `Content-Encoding: gzip`
- Browser decompresses transparently

**Example impact:** An 11,000-entry JSON file:
- Without compression: ~26 MB
- With gzip: ~3.8 MB (~7× smaller)

---

## Persistent Connections (Keep-Alive)

HTTP/1.0: new TCP connection per request — slow and resource-intensive.

HTTP/1.1: **persistent connections by default** — one TCP connection reused for multiple requests.

- `Connection: keep-alive` — keep connection open
- `Connection: close` — close after response (HTTP/1.0 default, can be forced in HTTP/1.1)

Options on keep-alive:
- `timeout` — how long to keep open
- `max` — max requests before closing

---

## Handling Large Requests and Responses

### Large Requests — Multipart

Used for file uploads. Binary data is split into **parts** separated by a boundary delimiter.

```
Content-Type: multipart/form-data; boundary=<delimiter>
```

- Each part is separated by the boundary value in the body
- Server reads and reassembles the parts

### Large Responses — Chunked / Server-Sent Events

Server streams data to client in chunks instead of one large payload:

```
Content-Type: text/event-stream
Connection: keep-alive
```

- Server keeps the connection open and sends data chunks as events
- Client appends each chunk, building the full response progressively

---

## SSL / TLS / HTTPS

| Term | What it is |
|------|-----------|
| **SSL** | Original encryption protocol — now **deprecated** (security vulnerabilities) |
| **TLS** | Modern replacement for SSL — encrypts data in transit, uses certificates to authenticate the server. Current recommended: **TLS 1.3** |
| **HTTPS** | HTTP + TLS — same HTTP protocol but with TLS encryption layer on top |

TLS prevents:
- Eavesdropping (data interception)
- Tampering (data modification in transit)
- Protocol downgrade attacks (HSTS header enforces this)

> For backend application-layer work, knowing HTTPS = HTTP + TLS encryption is sufficient. TLS handshake internals are network engineering territory.

---

## Summary — What You Need to Internalize

| Concept | Key takeaway |
|---------|-------------|
| Statelessness | Every request is self-contained; state is managed via cookies/tokens |
| Client-server | Client always initiates; server always responds |
| Headers | Metadata that controls behavior — caching, auth, format, security |
| Methods | Define intent — GET/POST/PATCH/PUT/DELETE + idempotency |
| CORS | Browser safety mechanism; pre-flight fires for JSON/non-simple requests |
| Status codes | Universal language for request outcome — know 2xx/3xx/4xx/5xx |
| Caching | ETag + Cache-Control + conditional requests → avoid re-downloading |
| Content negotiation | Client declares preferences; server responds accordingly |
| Compression | gzip cuts large payloads dramatically |
| Keep-alive | Default in HTTP/1.1 — reuse connections |
| Multipart / streaming | Patterns for large file upload/download |
| HTTPS/TLS | Encryption layer over HTTP — mandatory in production |
