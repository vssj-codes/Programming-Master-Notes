# Table-of-Contents

<!-- toc -->

- [Serialization and Deserialization for Backend Engineers](#serialization-and-deserialization-for-backend-engineers)
  * [The Core Problem](#the-core-problem)
  * [The Solution — A Common Standard](#the-solution--a-common-standard)
  * [Where Does This Sit in the OSI Model?](#where-does-this-sit-in-the-osi-model)
  * [Types of Serialization Standards](#types-of-serialization-standards)
    + [Text-Based](#text-based)
    + [Binary](#binary)
  * [JSON (JavaScript Object Notation)](#json-javascript-object-notation)
    + [Structure Rules](#structure-rules)
    + [Why JSON?](#why-json)
  * [The Flow in Practice](#the-flow-in-practice)
    + [Client sends a POST request:](#client-sends-a-post-request)
    + [Server processes and responds:](#server-processes-and-responds)
  * [Series Approach — Most Popular Choices](#series-approach--most-popular-choices)
  * [Summary](#summary)

<!-- tocstop -->

---

# Serialization and Deserialization for Backend Engineers

**Source:** Sriniously — Backend from First Principles (Video 7)
**Link:** [Watch](https://www.youtube.com/watch?v=vzg90tY3uM0)

---

## The Core Problem

A JavaScript client needs to send data to a Rust server. JavaScript and Rust have completely different type systems — JS is dynamic/interpreted, Rust is strict/compiled. How does data sent by one make sense to the other?

The same problem applies to the response: how does the client parse what the server sends back?

---

## The Solution — A Common Standard

Both sides agree on a **serialization standard** — a common format for encoding data during transmission.

```
Client (JS)                         Server (Rust)
  JS object  →  serialize to JSON  →  network  →  deserialize to Rust struct
  JS object  ←  deserialize JSON   ←  network  ←  serialize from Rust struct
```

**Serialization** = converting language-specific data → common format
**Deserialization** = converting common format → language-specific data

> The whole point: data becomes **language-agnostic** during transmission, so any client and any server can communicate regardless of their tech stack.

---

## Where Does This Sit in the OSI Model?

- Backend engineers work at **Layer 7 (Application Layer)**
- At this layer, the format is JSON (or whatever standard you chose)
- Below that, data gets converted into frames → IP packets → bits → voltage signals over fiber — **not your concern as a backend engineer**
- On the receiving end, data reconverts back up to JSON at the application layer

**Mental model:** Client serializes to JSON → *magic network stuff* → Server receives JSON. That's all you need to care about.

---

## Types of Serialization Standards

### Text-Based
| Format | Notes |
|--------|-------|
| **JSON** | Most popular for HTTP/REST (~80% of use cases). Human-readable. |
| YAML | Used in config files (Docker, K8s, CI/CD) |
| XML | Legacy, still used in SOAP APIs and some enterprise systems |

### Binary
| Format | Notes |
|--------|-------|
| **Protobuf** | Most popular binary format, used with gRPC |
| Avro | Used in data streaming (Kafka) |
| MessagePack | Compact alternative to JSON |

> For this series: **JSON** — the dominant standard for HTTP client-server communication.

---

## JSON (JavaScript Object Notation)

### Structure Rules

```json
{
  "id": 1,
  "title": "Some Book",
  "author": "Jane Doe",
  "published": true,
  "tags": ["backend", "http"],
  "address": {
    "country": "India",
    "phone": 3456
  }
}
```

1. Starts and ends with curly braces `{}`
2. **Keys must be strings in double quotes** — no single quotes, no unquoted keys
3. Values can be: **string**, **number**, **boolean**, **null**, **array**, or **nested object**
4. Nested objects follow the same rules recursively

### Why JSON?

- **Human-readable** — you can look at it and understand the data
- **Fundamental data types** — maps naturally to most programming languages
- **Ubiquitous** — used in HTTP transmission, config files, logging, storage

---

## The Flow in Practice

### Client sends a POST request:
```
POST /api/books HTTP/1.1
Content-Type: application/json

{
  "id": 1,
  "title": "Some Book",
  "author": "Jane Doe"
}
```

### Server processes and responds:
```
HTTP/1.1 201 Created
Content-Type: application/json

{
  "data": [
    { "id": 1, "title": "Some Book", "author": "Jane Doe" },
    { "id": 2, "title": "Another Book", "author": "John Smith" }
  ]
}
```

1. Client serializes JS object → JSON string
2. JSON transmitted over HTTP
3. Server deserializes JSON → Rust struct (or Python dict, Go struct, etc.)
4. Server performs business logic
5. Server serializes response → JSON string
6. Client deserializes JSON → JS object → renders UI

---

## Series Approach — Most Popular Choices

The backend domain is huge. This series focuses on the most widely used technology in each category:

| Category | Choice | Why |
|----------|--------|-----|
| Communication | HTTP / REST | Most common client-server protocol |
| Database | PostgreSQL | Most popular relational DB for startups and enterprise |
| Serialization | JSON | Dominant format for HTTP communication |

> Master these first. You can learn gRPC, MongoDB, Protobuf, etc. along the way as your career progresses.

---

## Summary

Serialization and deserialization is the mechanism that enables **cross-language, cross-platform communication**. Both sides agree on a standard format (JSON), convert their native data structures to/from that format, and the data remains understandable regardless of the tech stack on either end.
