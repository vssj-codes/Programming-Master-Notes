# Table-of-Contents

<!-- toc -->

- [Validations and Transformations for Backend Engineers](#validations-and-transformations-for-backend-engineers)
  * [Where Validations and Transformations Happen](#where-validations-and-transformations-happen)
    + [Quick Recap of Layers](#quick-recap-of-layers)
  * [Why Validate?](#why-validate)
    + [What Happens Without Validation](#what-happens-without-validation)
    + [What Happens With Validation](#what-happens-with-validation)
  * [Three Types of Validation](#three-types-of-validation)
    + [1. Syntactic Validation](#1-syntactic-validation)
    + [2. Semantic Validation](#2-semantic-validation)
    + [3. Type Validation](#3-type-validation)
  * [Complex Validation](#complex-validation)
  * [Transformation](#transformation)
    + [Examples](#examples)
    + [Type Casting (a special case of transformation)](#type-casting-a-special-case-of-transformation)
  * [Frontend Validation vs Backend Validation](#frontend-validation-vs-backend-validation)
    + [Why You Need Both](#why-you-need-both)
  * [Summary](#summary)

<!-- tocstop -->

---

# Validations and Transformations for Backend Engineers

**Source:** Sriniously — Backend from First Principles (Video 9)
**Link:** [Watch](https://www.youtube.com/watch?v=qedj_JjjL-U)

---

## Where Validations and Transformations Happen

In a typical backend architecture:

```
Client → Route Matching → [Validation & Transformation] → Controller → Service → Repository → Database
```

The pipeline sits **after route matching** and **before any business logic** — at the entry point of the controller layer, before calling service methods.

### Quick Recap of Layers

| Layer | Responsibility |
|-------|---------------|
| **Controller** | HTTP concerns — status codes, response format, calling service methods |
| **Service** | Business logic — calls repository methods, sends emails/notifications, webhooks |
| **Repository** | Database operations — queries, insertions, deletions |

---

## Why Validate?

All data from the client — JSON body, query parameters, path parameters, headers — must be verified before any significant logic runs.

### What Happens Without Validation

1. Client sends `{ "name": 0 }` instead of `{ "name": "some string" }`
2. No validation → data flows through controller → service → repository
3. Repository tries to INSERT `0` into a `TEXT` column in PostgreSQL
4. Database rejects it (type constraint) → unhandled error
5. Client gets `500 Internal Server Error` — poor UX, no useful feedback

### What Happens With Validation

1. Client sends `{ "name": 0 }`
2. Validation pipeline catches: expected `string`, received `number`
3. Client gets `400 Bad Request` with clear error: "name must be a string"
4. No unnecessary database call, no server crash

> Validation errors also serve as informal API documentation — sending an empty payload tells you exactly what fields are required and what constraints they have.

---

## Three Types of Validation

### 1. Syntactic Validation

Does the data follow a required **structural format**?

| Field | Structure check |
|-------|----------------|
| Email | `<local>@<domain>.<tld>` |
| Phone | Country code + digit count |
| Date | `YYYY-MM-DD` or other expected format |
| URL | `https://...` |

Example: `"test"` fails email validation, `"test@gmail.com"` passes.

### 2. Semantic Validation

Does the data **make sense** in context?

| Field | Semantic check |
|-------|---------------|
| Date of birth | Cannot be in the future |
| Age | Must be 1–120 (reasonable human age) |
| End date | Must be after start date |
| Quantity | Cannot be negative |

Example: `"2026-06-12"` as date of birth → rejected ("cannot be in the future").

### 3. Type Validation

Does the data match the expected **data type**?

| Field | Expected | Received | Result |
|-------|----------|----------|--------|
| `stringField` | string | string | Pass |
| `numberField` | number | string | Fail |
| `arrayField` | array of strings | array of numbers | Fail |
| `booleanField` | boolean | string | Fail |

Type validation can go deep — e.g. "each element of the array must be a string."

---

## Complex Validation

Validation constraints can depend on relationships between fields:

**Cross-field matching:**
- `password` and `passwordConfirmation` must have the same value
- Password must be at least 8 characters

**Conditional requirements:**
- If `married` is `true`, then `partner` (name) is **required**
- If `married` is `false`, `partner` is optional

These are defined in the same validation pipeline, keeping all input logic in one place.

---

## Transformation

Converting client data into the format the service layer expects — **before or after validation**.

### Examples

| Input (from client) | Transformation | Output (to service) |
|---------------------|---------------|---------------------|
| `"TeSt@Gmail.COM"` | Lowercase email | `"test@gmail.com"` |
| `"1234567890"` (phone) | Prepend country code | `"+1234567890"` |
| `"2025-06-12"` (date) | Reformat | `"12/06/2025"` |

### Type Casting (a special case of transformation)

Query parameters arrive as **strings by default** — always, regardless of their logical type.

```
GET /bookmarks?page=2&limit=20
```

Server receives: `page = "2"`, `limit = "20"` (strings)

Without transformation:
- Validation expects `number` → receives `string` → fails with "expected number, received string"

With transformation:
- Cast `"2"` → `2`, `"20"` → `20` → then validate (> 0, < 500, etc.)

> Validation and transformation are paired into a **single pipeline** so all input data logic lives in one place.

---

## Frontend Validation vs Backend Validation

| | Frontend Validation | Backend Validation |
|---|---|---|
| **Purpose** | User experience (UX) | Security + data integrity |
| **Provides** | Immediate feedback before API call | Authoritative rejection of bad data |
| **Can be bypassed?** | Yes (Postman, curl, modified JS) | No — it's the last line of defense |
| **Required?** | Nice to have | **Mandatory** |

### Why You Need Both

- A server can have many clients: web app, mobile app, Postman, CLI tools, third-party integrations
- Frontend validation only protects one client's UX
- If the backend relies on frontend validation, the server **breaks when the client changes** or when accessed directly via API

> **Rule:** Design server-side validation as if there is no frontend. Be as strict and specific as possible. Frontend validation is a UX convenience, not a security measure.

---

## Summary

| Concept | What it means |
|---------|--------------|
| **Validation** | Confirm client data matches expected format, type, and meaning before any business logic |
| **Transformation** | Convert client data into the format the service layer needs (lowercasing, type casting, reformatting) |
| **Pipeline** | Both happen together at the entry point, after route matching, before controller logic |
| **Syntactic** | Structural format checks (email, phone, date patterns) |
| **Semantic** | Logical sense checks (date not in future, age within range) |
| **Type** | Data type matching (string, number, boolean, array) |
| **Frontend vs backend** | Frontend = UX feedback; backend = security + integrity. Always do both. |
