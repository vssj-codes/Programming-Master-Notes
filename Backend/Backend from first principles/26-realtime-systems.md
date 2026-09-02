# Table-of-Contents

<!-- toc -->

- [Real-Time Systems](#real-time-systems)
  * [The Core Limitation](#the-core-limitation)
  * [Approach 1: Polling](#approach-1-polling)
  * [Approach 2: Long Polling](#approach-2-long-polling)
  * [Approach 3: Server-Sent Events (SSE)](#approach-3-server-sent-events-sse)
  * [Approach 4: WebSockets — Bidirectional Communication](#approach-4-websockets--bidirectional-communication)
    + [The Handshake](#the-handshake)
    + [Frame Format](#frame-format)
    + [Masking — Proxy Cache Poisoning Defense](#masking--proxy-cache-poisoning-defense)
    + [Ping / Pong — Heartbeats](#ping--pong--heartbeats)
  * [Connection Limits](#connection-limits)
    + [Limit 1: File Descriptors](#limit-1-file-descriptors)
    + [Limit 2: Client-Side Port Range](#limit-2-client-side-port-range)
    + [Limit 3: Memory](#limit-3-memory)
  * [The Multi-Instance Problem](#the-multi-instance-problem)
  * [The Solution: Pub/Sub Between Instances](#the-solution-pubsub-between-instances)
    + [Delivery Guarantees — Choose Carefully](#delivery-guarantees--choose-carefully)
  * [Catching Up After Disconnection](#catching-up-after-disconnection)
  * [Fan-Out at Scale](#fan-out-at-scale)
  * [Reconnection Storm](#reconnection-storm)
  * [Comparison Table](#comparison-table)
  * [Summary](#summary)

<!-- tocstop -->

---

# Real-Time Systems

**Source:** Sriniously — Backend from First Principles (Video 26)
**Link:** [Watch](https://www.youtube.com/watch?v=estH64OkwxU)

---

## The Core Limitation

Everything built so far follows one shape: **the client asks, the server answers**. The server has no mechanism to speak first — it can only reply to a request the client has already made.

If the server has new information (a task moved on a shared board), there is no channel to push that to connected clients. The client must ask.

---

## Approach 1: Polling

**How it works:** Browser sends a request every N seconds — "has anything changed on board 412?"

Most responses: "No." Occasionally: "Yes — here's the change."

**Problems:**

| Problem | Detail |
|---------|--------|
| **Inherent delay** | Average delay = interval / 2. At 3s polling, users see changes ~1.5s late on average |
| **Cost scales with users, not events** | 10,000 open boards × 1 poll/3s = 3,333 DB queries/second, almost all returning "no change" |
| **Empty boards cost the same** | A board with zero events costs identical compute to a board with constant activity |
| **Mobile battery drain** | Each poll wakes the phone's radio — one of the most expensive hardware operations on a mobile device |

Reducing the interval to 1s makes the UX 3× better but makes the infrastructure bill 3× worse.

> **Rule:** Cost of polling scales with the number of users, not the number of events. You pay for the medium, not the value.

**When polling is fine:** Internal tooling with a handful of users. Not for user-facing products at scale.

---

## Approach 2: Long Polling

**How it works:** Client sends a request. Server holds it open (up to ~60s) without responding. The moment something changes, the server writes and sends the response. Client immediately opens the next long-poll.

Documented in **RFC 6202** along with its shortcomings.

**Problems:**
- After the server responds, the client must process the response and open a fresh request
- During that gap, the server has no live connection to the client
- RFC 6202: average latency ≈ 1 network transit; worst case > 3 due to the gap

Long polling was widely used for years but carries the same fundamental issue: the server cannot sustain a continuous channel.

---

## Approach 3: Server-Sent Events (SSE)

**Key insight:** What if the response just never ends?

**How it works:**
1. Browser makes a normal HTTP GET request
2. Server replies `200 OK` with `Content-Type: text/event-stream`
3. Server never closes the connection
4. Whenever data is available, the server writes text into the open response body and flushes
5. Connection stays open until the browser tab closes or the network drops

**Event format** (four fields):

```
id: 42
event: task.moved
data: {"taskId":"abc","from":"todo","to":"in-progress","boardId":"412"}
retry: 3000
```

| Field | Purpose |
|-------|---------|
| `id` | Unique sequence ID; client sends back on reconnect so server can replay missed events |
| `event` | Event type name (e.g., `task.moved`, `task.deleted`) |
| `data` | Payload — client uses this to update the UI directly |
| `retry` | How long the browser should wait before reconnecting (in ms) |

**Browser handles reconnection automatically** — it's in the WHATWG spec. On reconnect, it sends a `Last-Event-ID` header with the ID of the last received event. No client-side reconnect code needed.

**SSE is not a toy.** Real production usage:
- **Uber's driver push platform** — built on SSE
- **LinkedIn** — instant messaging, typing indicators, read receipts
- **Every major LLM API** — token-by-token streaming (each `data:` field = one token)

**Limitation:** **One direction only.** The server can stream continuously to the browser, but the browser cannot send data back on the same connection. A task drag would require a separate POST request.

---

## Approach 4: WebSockets — Bidirectional Communication

WebSocket provides a **persistent, bidirectional stream** on a single connection.

### The Handshake

WebSocket starts as a normal HTTP GET request — it travels through every proxy and load balancer that speaks HTTP:

```
Client → GET /ws HTTP/1.1
         Upgrade: websocket
         Sec-WebSocket-Key: <16 random bytes, base64 encoded>

Server → HTTP/1.1 101 Switching Protocols
         Upgrade: websocket
         Sec-WebSocket-Accept: <SHA1(key + fixed_constant), base64>
```

The `Sec-WebSocket-Accept` value proves the server understood it was a WebSocket request and not a cached replay of a prior `200` response. After `101`, HTTP ends and a bidirectional binary frame stream begins.

### Frame Format

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-------+-+-------------+-------------------------------+
|F|R|R|R| opcode|M| Payload len |    Extended payload length    |
|I|S|S|S|  (4)  |A|     (7)    |             (if any)          |
|N|V|V|V|       |S|             |                               |
| |1|2|3|       |K|             |                               |
+-+-+-+-+-------+-+-------------+-------------------------------+
```

| Field | Values / Meaning |
|-------|-----------------|
| **FIN** | 1 = last fragment of a message |
| **Opcode** | 1=text, 2=binary, 8=close, 9=ping, 10=pong, 0=continuation |
| **MASK** | 1 = payload is masked (always set for client→server frames) |
| **Payload length** | <126 bytes: stored in 7 bits (2-byte total header); up to 65535: 2 extra bytes; larger: 8 extra bytes |

**Comparison:** A tiny WebSocket frame has a 2-byte header vs. hundreds of bytes of HTTP headers per polling request.

### Masking — Proxy Cache Poisoning Defense

Every client→server frame has its payload XOR'd with a random 4-byte key (sent in plaintext, NOT encryption). The masking key changes per frame.

**Why:** An experiment showed that without masking, a malicious page could open a WebSocket to an attacker-controlled server, send bytes crafted to look like a valid HTTP response for a popular resource (e.g., Google Analytics), and trick an intermediate caching proxy into caching the attacker's payload. Anyone behind that proxy would then receive the attacker's file when requesting the real one.

**Masking defeats this** because the page's JavaScript cannot predict the random key, so it cannot choose the bytes that appear on the wire after XOR.

> Masking protects the network's proxy infrastructure, not the data itself.

### Ping / Pong — Heartbeats

Either side can send `opcode 9` (ping); the other **must** respond with `opcode 10` (pong).

**Why this matters:** A dead connection and an idle connection look identical to TCP — no bytes arrive in either case. Without heartbeats, a server would hold a socket for a dead client indefinitely, wasting memory.

**Production rule:** If a ping receives no pong within 30 seconds → connection is dead → close the socket and free resources.

---

## Connection Limits

### Limit 1: File Descriptors

In Unix, everything is a file — including network sockets. Each open connection consumes one file descriptor.

```bash
ulimit -n   # default: 1024
```

| Runtime | File descriptor limit |
|---------|----------------------|
| Python, Node, Java | OS default (~1024 soft limit) |
| **Go** | Go runtime raises its own limit to ~148,575 at startup (uses epoll, not select) |

Go's standard library explicitly raises the soft limit because it doesn't use `select` (which has a hard-coded `FD_SET` size limit). The runtime sets the limit to `max - 1` intentionally to detect external changes.

### Limit 2: Client-Side Port Range

A TCP connection is identified by 4 values: `(src_ip, src_port, dst_ip, dst_port)`. The server's address and port are fixed for all connections — so uniqueness comes entirely from the client's source port.

```bash
cat /proc/sys/net/ipv4/ip_local_port_range
# → 32768 60999   (28,232 available ports)
```

A single client IP can open ~28,000 connections to one server before exhausting ephemeral ports.

**Fix:** Dial from multiple source IPs. Three IPs × 25,000 connections each = 75,000 concurrent connections with zero failures.

**Implication for load testing:** You need multiple machines (or multiple IPs per machine) to generate realistic load — a single machine cannot simulate more than ~28,000 concurrent users against one server.

### Limit 3: Memory

Measured on a Go WebSocket server (no application logic, just holding connections):

| Metric | Value |
|--------|-------|
| Heap per connection | ~9,750 bytes |
| 1 million connections | ~9.75 GB heap (before kernel's own socket structures) |
| 61 GB machine capacity | ~1 million connections easily achievable |

Most of that 9.75 KB is the goroutine and its buffers — not the raw socket. At extreme scale, people use a single epoll watcher for all sockets instead of one goroutine per connection to reduce per-connection overhead.

---

## The Multi-Instance Problem

A WebSocket connection is **stateful** — it lives in one specific process's memory. Instance 1 holds Browser A's socket. Instance 2 holds Browser B's socket. When Browser A moves a task, Instance 1 needs to notify Browser B — but it has no knowledge of Instance 2's connections.

**Sticky sessions** (routing a client to the same instance always) do not solve this — Browser B is still on Instance 2 and knows nothing about Browser A's action.

---

## The Solution: Pub/Sub Between Instances

```
Browser A moves task
        │
        ▼
   Instance 1
   (receives the change)
        │
        ▼ publish("board:412", event)
   [Redis / NATS / Kafka]
        │
        ├──► Instance 1 (subscribed to board:412) → writes frame to Browser A's socket
        └──► Instance 2 (subscribed to board:412) → writes frame to Browser B's socket
```

The instance that **receives** the change and the instance that **delivers** the notification can be on different machines.

### Delivery Guarantees — Choose Carefully

| Tool | Model | Behavior |
|------|-------|----------|
| **Redis Pub/Sub** | At-most-once (fire and forget) | If an instance is not subscribed at the exact moment of publish, the message is gone forever |
| **Redis Streams / Kafka / RabbitMQ** | At-least-once | Messages are stored; instances can acknowledge; replay is possible |

**Redis Pub/Sub failure scenario:** A container restarts — in the seconds between the old container going down and the new one subscribing, any published messages for users that instance was serving are permanently lost. Those users only see the update on next page refresh.

---

## Catching Up After Disconnection

Connections drop: phones switch networks, laptops close, load balancers hit idle timeouts. A production-grade real-time system must handle reconnection gaps.

**Pattern (same as SSE's `Last-Event-ID`):**

1. Server assigns a monotonically increasing **sequence number** to every message
2. Client tracks the last sequence number it received
3. On reconnect, client sends: "I last received sequence 2"
4. Server replays everything from sequence 3 onward

Uber's push platform works exactly this way. The client connects with `sequence: 0`; if the connection breaks, it reconnects with its last confirmed sequence number and the server picks up from there.

---

## Fan-Out at Scale

When 30,000 users are watching the same board, one event = 30,000 individual socket writes.

**Discord's measurement:** Publishing a single event to a 30,000-member community took **900ms to 2,100ms** — a serious problem for a messaging platform.

**Discord's fix:** Instead of one process doing all 30,000 writes, distribute the fan-out across the machines that own those connections. Each machine only writes to its own sockets.

---

## Reconnection Storm

When a container holding 50,000 connections dies, all 50,000 clients attempt to reconnect simultaneously. Each reconnect triggers an initial state fetch (current board state, active users, etc.) — 50,000 expensive queries in under a second.

**Slack's fix:** Cache the initial state payload at the network edge. On reconnection, clients get initial state from the edge cache, not the origin. The origin only sees reconnection load when the cache misses.

---

## Comparison Table

| Technique | Direction | Protocol | Reconnect | Cost model | Use case |
|-----------|-----------|----------|-----------|------------|----------|
| **Polling** | Client→Server | HTTP | Client-managed | Per user × interval | Internal tools, infrequent updates |
| **Long polling** | Client→Server | HTTP | Client-managed | Per user (lower than polling) | Legacy real-time, simple setups |
| **Server-Sent Events** | Server→Client | HTTP | Browser-native (spec) | Per active connection | LLM streaming, notifications, dashboards, one-way feeds |
| **WebSockets** | Bidirectional | WS (after upgrade) | Application-managed | Per active connection | Collaborative tools, chat, live boards, gaming |

---

## Summary

| Concept | Key point |
|---------|----------|
| **HTTP limitation** | Client always initiates; server can only reply |
| **Polling cost** | Scales with users × interval, not with actual events |
| **Long polling** | Holds request open; gap on reconnect causes message loss risk |
| **SSE** | HTTP stream that never closes; browser auto-reconnects; one-way; powers all LLM token streaming |
| **WebSocket** | Starts as HTTP GET → upgrades to bidirectional frame stream |
| **Frame overhead** | 2-byte header for small messages vs. hundreds of bytes of HTTP headers |
| **Masking** | Protects network proxies from cache poisoning, not the data itself |
| **Ping/pong** | Heartbeat to detect dead connections (idle = dead from TCP's view) |
| **File descriptor limit** | Go auto-raises to ~148K; Python/Node stay at OS default ~1024 |
| **Port exhaustion** | One client IP can open ~28K connections; fix by dialing from multiple IPs |
| **Memory** | ~9.75 KB heap per connection in Go; 1M connections ≈ 10 GB |
| **Multi-instance** | Persistent connections are stateful; sticky sessions don't help |
| **Pub/Sub** | Redis (fire-and-forget) or Kafka/streams (at-least-once) for cross-instance delivery |
| **Sequence numbers** | Essential for catch-up after reconnect; same idea as SSE's Last-Event-ID |
| **Fan-out** | N subscribers × 1 event = N writes; distribute across machines at scale (Discord) |
| **Reconnection storm** | Cache initial state at the edge to absorb simultaneous reconnects (Slack) |
