# Table-of-Contents

<!-- toc -->

- [What is Routing in Backend?](#what-is-routing-in-backend)
  * [Routing = Mapping URL + Method → Server Logic](#routing--mapping-url--method-%E2%86%92-server-logic)
  * [Types of Routes](#types-of-routes)
    + [Static Routes](#static-routes)
    + [Dynamic Routes (Path/Route Parameters)](#dynamic-routes-pathroute-parameters)
  * [Query Parameters](#query-parameters)
    + [Why not use path parameters for everything?](#why-not-use-path-parameters-for-everything)
    + [Pagination Example](#pagination-example)
  * [Nested Routes](#nested-routes)
  * [Route Versioning and Deprecation](#route-versioning-and-deprecation)
    + [Why version?](#why-version)
    + [Deprecation workflow:](#deprecation-workflow)
  * [Catch-All Route](#catch-all-route)
  * [Summary](#summary)

<!-- tocstop -->

---

# What is Routing in Backend?

**Source:** Sriniously — Backend from First Principles (Video 6)
**Link:** [Watch](https://www.youtube.com/watch?v=SubuU1iOC2s)

---

## Routing = Mapping URL + Method → Server Logic

- **HTTP method** expresses the **what** (intent) — GET, POST, PUT, DELETE
- **Route / URL path** expresses the **where** (resource)
- The server takes both and maps them to a **handler** — a set of instructions that performs business logic, database operations, etc.

```
GET /api/books  →  Handler A (fetch all books)
POST /api/books →  Handler B (create a book)
```

Same route, different methods → different handlers. The combination of **method + route** forms a unique key that never clashes.

---

## Types of Routes

### Static Routes

Routes with no variable parts — the path string is constant.

```
GET  /api/books   → always returns list of books
POST /api/books   → always creates a book
```

Nothing changes in the route — same string every time, same kind of response.

### Dynamic Routes (Path/Route Parameters)

Routes with variable segments, marked by a **colon prefix** (`:id`) — this is a universal convention across languages (Node.js, Python, Go, Rust, Java, Ruby).

```
GET /api/users/:id
```

Request: `GET /api/users/123`
- Server matches the pattern, extracts `id = "123"`
- All path segments are treated as **strings** (even numbers)

Reads naturally: *"GET me the user whose ID is 123"*

> These variable segments are called **path parameters** or **route parameters** — they go directly in the URL path after a `/`.

---

## Query Parameters

Key-value pairs appended after `?` in the URL, used to send metadata in **GET requests** (which have no body).

```
GET /api/search?query=some+value
GET /api/books?page=2&limit=20&sort=title&order=asc
```

### Why not use path parameters for everything?

Path parameters serve a **semantic purpose** — they identify a resource:
```
/api/users/123          ← "the user with ID 123"
```

Sending arbitrary data (search terms, pagination, filters) in path params would look like:
```
/api/search/some value  ← breaks semantics, hard to maintain
```

Query params exist for **non-resource-identifying data**: filters, sorting, pagination, search terms.

### Pagination Example

```
GET /api/books
```
Response:
```json
{
  "data": [...20 books...],
  "total": 100,
  "currentPage": 1,
  "totalPages": 5,
  "limit": 20
}
```

Next page:
```
GET /api/books?page=2
```

Common query params: `page`, `limit`, `sort`, `order`, `filter`, `search`, `q`

---

## Nested Routes

Nesting resources to express **hierarchical relationships** — standard REST practice.

```
GET /api/users                    → all users
GET /api/users/123                → user 123
GET /api/users/123/posts          → all posts by user 123
GET /api/users/123/posts/456      → post 456 by user 123
```

Each level adds semantic meaning. You can stop at any depth — each is a valid, distinct route mapped to its own handler.

---

## Route Versioning and Deprecation

Adding a version segment (`v1`, `v2`) to manage breaking API changes.

```
GET /api/v1/products  →  { data: [{ id, name, price }] }
GET /api/v2/products  →  { data: [{ id, title, price }] }
```

### Why version?

- New requirements (e.g. serving a mobile app) may need a different response structure
- Instead of changing the route (`/api/new-products`) or breaking existing clients, version it
- V1 and V2 coexist → clients get a migration window
- Eventually deprecate V1, promote V2

### Deprecation workflow:
1. Release V2 alongside V1
2. Notify clients that V1 is deprecated
3. Give a migration window
4. Remove V1, optionally rename V2 → V1

---

## Catch-All Route

A fallback route (`/*` or `*`) placed **after all other routes** — catches any request that didn't match a defined route.

```
GET /api/v3/products  →  (no handler exists)
                      →  catch-all returns 404 with friendly message
```

Without a catch-all, the server returns a null/empty response (default behavior). With one, you send a user-friendly "route not found" message instead.

---

## Summary

| Concept | What it does |
|---------|-------------|
| **Static route** | Fixed path, no variables (`/api/books`) |
| **Dynamic route** | Path parameters with `:param` (`/api/users/:id`) |
| **Query parameters** | Key-value pairs after `?` for filters, search, pagination |
| **Nested route** | Hierarchical resource paths (`/users/:id/posts/:postId`) |
| **Versioning** | `v1`/`v2` segments for managing breaking changes |
| **Catch-all** | `/*` fallback for undefined routes → friendly 404 |
| **Route matching** | Method + path = unique key → mapped to a handler |
