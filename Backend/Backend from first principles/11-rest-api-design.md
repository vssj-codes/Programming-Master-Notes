# Table-of-Contents

<!-- toc -->

- [Complete REST API Design](#complete-rest-api-design)
  * [History — Where REST Came From](#history--where-rest-came-from)
    + [Tim Berners-Lee (1990)](#tim-berners-lee-1990)
    + [The Scalability Crisis](#the-scalability-crisis)
    + [Roy Fielding (~1993–2000)](#roy-fielding-1993%E2%80%932000)
    + [What "REST" Means](#what-rest-means)
  * [URL Structure for APIs](#url-structure-for-apis)
    + [Rules](#rules)
  * [Idempotency](#idempotency)
    + [POST as an Open-Ended Method](#post-as-an-open-ended-method)
  * [API Design Workflow](#api-design-workflow)
  * [CRUD Endpoints — The Pattern](#crud-endpoints--the-pattern)
    + [Create](#create)
    + [List All](#list-all)
    + [Get Single](#get-single)
    + [Update](#update)
    + [Delete](#delete)
    + [Route Similarity Pattern](#route-similarity-pattern)
  * [Custom Actions](#custom-actions)
  * [Pagination](#pagination)
    + [Query Parameters](#query-parameters)
    + [Response Structure](#response-structure)
    + [Example](#example)
  * [Sorting](#sorting)
    + [Query Parameters](#query-parameters-1)
  * [Filtering](#filtering)
  * [Sane Defaults](#sane-defaults)
  * [Response Code Cheat Sheet](#response-code-cheat-sheet)
  * [Consistency Rules](#consistency-rules)
    + [JSON Payloads](#json-payloads)
    + [Routes](#routes)
    + [Across the Entire API](#across-the-entire-api)
  * [Best Practices Summary](#best-practices-summary)

<!-- tocstop -->

---

# Complete REST API Design

**Source:** Sriniously — Backend from First Principles (Video 11)
**Link:** [Watch](https://www.youtube.com/watch?v=RG6q57DwV8Y)

---

## History — Where REST Came From

### Tim Berners-Lee (1990)
Built the World Wide Web to share knowledge globally. Within ~1 year invented:
URI, HTTP, HTML, first web server, first web browser, first WYSIWYG HTML editor.

### The Scalability Crisis
The web grew exponentially — the original design couldn't handle the scale.

### Roy Fielding (~1993–2000)
Co-founder of Apache HTTP server. Proposed **six constraints** to make the web scalable:

| Constraint | Meaning |
|-----------|---------|
| **Client-Server** | Separate UI (client) from data/logic (server) — evolve independently |
| **Uniform Interface** | Standardized way for components to communicate |
| **Layered System** | Hierarchical layers — each layer only sees the one below (enables load balancers, proxies) |
| **Cacheable** | Responses must be labeled cacheable or non-cacheable |
| **Stateless** | Every request carries all info needed — server stores no client context |
| **Code on Demand** | (Optional) Server can send executable code (e.g. JS) to the client |

Fielding co-authored the **HTTP/1.1 specification** with Berners-Lee, then named the architecture **REST (Representational State Transfer)** in his 2000 PhD dissertation.

### What "REST" Means

- **Representational** — resources have formats (JSON, XML, HTML); same resource can have different representations for different clients
- **State** — current condition/attributes of a resource (e.g. shopping cart items + quantities + total)
- **Transfer** — movement of resource representations between client and server via HTTP methods

---

## URL Structure for APIs

```
https://api.example.com/v1/books/harry-potter?sort=name&order=asc
│       │                 │  │     │           │
scheme  subdomain+domain  │  resource (plural) │
                       version    slug      query params
```

### Rules

- **Subdomain:** `api.` prefix is the industry standard
- **Versioning:** `/v1/`, `/v2/` in the path
- **Resources:** Always **plural nouns** in lowercase — `/books`, `/organizations`, `/users`
  - Even when fetching a single resource: `/books/:id` (not `/book/:id`)
- **No spaces or underscores** in URLs — use **hyphens** for multi-word slugs: `harry-potter`
- **Forward slash `/`** represents a **hierarchical relationship** between resources
- **Lowercase only** in path segments — avoids case-mismatch issues across environments

---

## Idempotency

> Performing the same action multiple times has the **same effect** as performing it once.

| Method | Idempotent? | Why |
|--------|-------------|-----|
| `GET` | Yes | Fetching data causes no side effects on the server |
| `PUT` | Yes | Replacing A→B repeatedly still leaves B |
| `PATCH` | Yes | Updating a field to the same value repeatedly = same state |
| `DELETE` | Yes | First call deletes; subsequent calls = 404 but no new side effects |
| `POST` | **No** | Each call creates a new resource with a new ID |

### POST as an Open-Ended Method

When an action doesn't fit any CRUD category (not create/read/update/delete), use `POST` as a **custom action** endpoint. Example: `POST /send-email`, `POST /organizations/:id/archive`.

---

## API Design Workflow

```
1. Wireframes / Figma designs
   → Identify resources (nouns): organizations, projects, tasks, users, tags
2. Database schema design
   → Define tables, columns, relationships
3. API interface design
   → Design routes, payloads, responses BEFORE writing any code
```

> Design the API interface first in a tool like Swagger, Insomnia, or Postman — independently of programming language. A separate session just for interface design makes APIs more intuitive.

---

## CRUD Endpoints — The Pattern

For every resource, you typically need five CRUD endpoints. Using `organizations` as an example:

### Create

```
POST /organizations
```

- **Body:** JSON with client-provided fields (exclude server-generated: `id`, `createdAt`, `updatedAt`)
- **Response:** `201 Created` + the newly created entity

### List All

```
GET /organizations
```

- **Response:** `200 OK` + paginated data (see pagination below)
- Returns `200` with empty `data: []` if no results — **never 404 on a list endpoint**

### Get Single

```
GET /organizations/:id
```

- **Response:** `200 OK` + single entity
- If not found → `404 Not Found`

### Update

```
PATCH /organizations/:id
```

- **Body:** Partial fields to update (not the full entity — that's `PUT`)
- **Response:** `200 OK` + updated entity
- Use `PATCH` over `PUT` unless you specifically need full replacement

### Delete

```
DELETE /organizations/:id
```

- **Response:** `204 No Content` — empty body
- Subsequent calls → `404 Not Found`

### Route Similarity Pattern

| Operation | Route | Method |
|-----------|-------|--------|
| Create | `/organizations` | `POST` |
| List | `/organizations` | `GET` |
| Get single | `/organizations/:id` | `GET` |
| Update | `/organizations/:id` | `PATCH` |
| Delete | `/organizations/:id` | `DELETE` |

The server differentiates create vs list (and get vs update vs delete) by the **HTTP method**, not the route.

---

## Custom Actions

For operations that don't fit CRUD — use `POST` with the action name appended:

```
POST /organizations/:id/archive
POST /projects/:id/clone
```

The route follows the hierarchy: resource → specific entity → action name.

**Response codes vary by action:**
- Archive (no new resource) → `200 OK`
- Clone (creates a new resource) → `201 Created`

> Don't assume all `POST` calls return `201` — custom actions may return `200`.

---

## Pagination

Used in **list APIs** to avoid returning all data at once (performance, serialization cost, bandwidth).

### Query Parameters

| Param | Default | Purpose |
|-------|---------|---------|
| `page` | `1` | Which portion of data to return |
| `limit` | `10` or `20` | How many items per page |

### Response Structure

```json
{
  "data": [...],
  "total": 50,
  "page": 2,
  "totalPages": 5
}
```

- `data` — the current page's items
- `total` — total count in the database (independent of pagination)
- `page` — current page number
- `totalPages` — calculated from `ceil(total / limit)`

### Example

```
GET /organizations?limit=2&page=1  →  org5, org4
GET /organizations?limit=2&page=2  →  org3, org2
GET /organizations?limit=2&page=3  →  org1
```

---

## Sorting

### Query Parameters

| Param | Default | Purpose |
|-------|---------|---------|
| `sortBy` | `createdAt` | Field to sort by |
| `sortOrder` | `desc` | `asc` or `desc` |

> Always apply a default sort (typically `createdAt` descending) even if the client sends nothing — otherwise results return in random database order.

```
GET /organizations?sortBy=name&sortOrder=asc  →  org1, org2, org3, org4, org5
```

---

## Filtering

Pass field names as query parameters with the desired value:

```
GET /organizations?status=active
GET /organizations?status=archived&name=org4
```

Multiple filters can be combined. If no data matches → return `200` with `data: []` and `total: 0` — **not 404**.

---

## Sane Defaults

The server should set reasonable defaults so clients don't need to send obvious parameters:

| Parameter | Default |
|-----------|---------|
| `page` | `1` |
| `limit` | `10` or `20` |
| `sortBy` | `createdAt` |
| `sortOrder` | `desc` |
| `status` (on create) | `active` |

> For `POST` calls: only require the minimum info needed. If `status` makes sense as `active` by default, don't force the client to send it.

---

## Response Code Cheat Sheet

| Scenario | Code |
|----------|------|
| Successful fetch / update | `200 OK` |
| Resource created | `201 Created` |
| Successful delete (no body) | `204 No Content` |
| Custom action (no creation) | `200 OK` |
| Custom action (creates resource) | `201 Created` |
| List API with no results | `200 OK` + empty array |
| Single resource not found | `404 Not Found` |

---

## Consistency Rules

### JSON Payloads
- Field names in **camelCase** — `organizationId`, `createdAt`, `sortOrder`
- Same field name across all resources — if `description` is used in organizations, don't use `desc` in projects

### Routes
- Always plural nouns: `/organizations`, `/projects`, `/tasks`
- Consistent structure across all resources

### Across the Entire API
- Same pagination format, same sorting params, same error format
- Once you establish a pattern in one endpoint, follow it everywhere
- Don't make consumers guess — if they integrated one resource, they should already know how the next one works

---

## Best Practices Summary

1. **Design before coding** — use Swagger/Insomnia/Postman to design the interface first
2. **Interactive documentation** — integrate Swagger/OpenAPI for testing and docs
3. **Intuitive and consistent** — routes, payloads, responses, naming — single standard throughout
4. **Sane defaults** — don't require obvious parameters
5. **No abbreviations** — `description` not `desc`, `organization` not `org`
6. **PATCH over PUT** — unless you need full replacement
7. **POST for custom actions** — anything outside CRUD
8. **Plural nouns** — always, even for single-resource endpoints
9. **Never 404 on list APIs** — return empty array with `200`
10. **Query params should be optional** — especially in list APIs
