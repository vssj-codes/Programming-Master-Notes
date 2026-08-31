# Table-of-Contents

<!-- toc -->

- [Authentication and Authorization for Backend Engineers](#authentication-and-authorization-for-backend-engineers)
  * [The Two-Sentence Summary](#the-two-sentence-summary)
  * [Historical Context of Authentication](#historical-context-of-authentication)
    + [Key Milestones](#key-milestones)
  * [Three Core Components of Modern Auth](#three-core-components-of-modern-auth)
    + [1. Sessions](#1-sessions)
    + [2. JWTs (JSON Web Tokens)](#2-jwts-json-web-tokens)
    + [3. Cookies](#3-cookies)
  * [Types of Authentication](#types-of-authentication)
    + [1. Stateful Authentication (Session-Based)](#1-stateful-authentication-session-based)
    + [2. Stateless Authentication (JWT-Based)](#2-stateless-authentication-jwt-based)
    + [3. API Key Authentication](#3-api-key-authentication)
    + [4. OAuth 2.0 + OpenID Connect (OIDC)](#4-oauth-20--openid-connect-oidc)
      - [The Delegation Problem](#the-delegation-problem)
      - [OAuth 1.0 (2007) — The Revolution](#oauth-10-2007--the-revolution)
      - [OAuth 2.0 (2010) — Simplified](#oauth-20-2010--simplified)
      - [OpenID Connect (OIDC, ~2014) — Adding Authentication](#openid-connect-oidc-2014--adding-authentication)
  * [When to Use Which Authentication Type](#when-to-use-which-authentication-type)
  * [Authorization — Role-Based Access Control (RBAC)](#authorization--role-based-access-control-rbac)
    + [How RBAC Works](#how-rbac-works)
    + [RBAC Flow](#rbac-flow)
  * [Security Considerations](#security-considerations)
    + [1. Generic Error Messages in Auth](#1-generic-error-messages-in-auth)
    + [2. Timing Attacks](#2-timing-attacks)

<!-- tocstop -->

---

# Authentication and Authorization for Backend Engineers

**Source:** Sriniously — Backend from First Principles (Video 8)
**Link:** [Watch](https://www.youtube.com/watch?v=A95rliroC8Q)

---

## The Two-Sentence Summary

- **Authentication** = *Who are you?* — assigning/verifying an identity in a given context
- **Authorization** = *What can you do?* — determining permissions/capabilities in that context

---

## Historical Context of Authentication

| Era | Mechanism | Principle |
|-----|-----------|-----------|
| Pre-industrial | Village elder vouches, handshake deals | Implicit trust (human recognition) |
| Medieval | Wax seals on documents | **Something you have** (possession) |
| Industrial Revolution | Telegraph pass-phrases | **Something you know** (shared secrets) |
| 1961 — MIT CTSS | First digital passwords (stored in plain text) | Something you know (digital) |
| 1970s | Diffie-Hellman key exchange, asymmetric cryptography, Kerberos | Public key infrastructure, ticket-based auth |
| 1990s | MFA (multi-factor authentication) | Combining: something you **know** + **have** + **are** |
| 2007 | OAuth 1.0 | Token-based delegation |
| 2010 | OAuth 2.0 | Bearer tokens, multiple grant flows |
| 2014 | OpenID Connect (OIDC) | Identity layer on top of OAuth 2.0 |

### Key Milestones

- **Plain text password incident (MIT 1961)** — someone printed the password file → birth of secure password storage → hashing
- **Diffie-Hellman (1970s)** — asymmetric cryptography became the backbone of all modern auth protocols
- **MFA** — combines: passwords/PINs (know) + smart cards/OTP (have) + biometrics (are)

---

## Three Core Components of Modern Auth

### 1. Sessions

HTTP is **stateless by design** — every request is isolated, no memory of past exchanges. This was fine for static pages but broke down for dynamic content (shopping carts, logged-in users).

**Sessions solve this** by creating temporary server-side context per user.

**How it works:**
1. User logs in → server generates a unique **session ID**
2. Server stores session ID + user data (role, cart, auth status) in a **persistent store** (Redis / database)
3. Server sends the session ID to the client as a **cookie**
4. Every subsequent request includes the cookie → server looks up session ID → identifies user
5. Sessions are **short-lived** — expire after a set time (e.g. 15 minutes), then re-created

**Storage evolution:**
- File-based → database-backed → **distributed in-memory stores** (Redis, Memcached)
- Redis preferred for fast read times

### 2. JWTs (JSON Web Tokens)

By mid-2000s, stateful sessions hit scalability limits:
- **Memory** — storing session data for millions of users is costly
- **Replication** — syncing session data across distributed servers introduces latency

**JWTs solve this** with a **stateless, self-contained token**.

**Structure** (three Base64-encoded parts separated by dots):

```
header.payload.signature
```

| Part | Contains |
|------|----------|
| **Header** | Metadata — signing algorithm (`alg`), token type (`typ`) |
| **Payload** | User data — `sub` (user ID), `iat` (issued at), `name`, `role`, etc. |
| **Signature** | Cryptographic signature using server's secret key — verifies integrity |

Inspect JWTs at [jwt.io](https://jwt.io)

**Advantages:**
- **Stateless** — no server-side storage needed
- **Scalable** — any server with the secret key can verify (perfect for microservices)
- **Portable** — lightweight, URL-friendly, can be stored in cookies or headers

**Disadvantages:**
- **Token theft** — if someone gets your JWT, they can impersonate you until it expires
- **Revocation is hard** — no way to invalidate a single JWT without changing the secret key (which logs out ALL users)

**Hybrid approach** — verify JWT stateless-ly, then check a **blacklist** in Redis for revoked tokens. Trades some statelessness for revocation capability.

> **Practical advice:** For production systems, use an auth provider (Auth0, Clerk, etc.) — they handle all the complexity of algorithms, hashing, salting, and security edge cases. Implement your own only for learning purposes.

### 3. Cookies

A cookie is a piece of data that a **server stores in the client's browser**.

Key properties:
- Set by the server via `Set-Cookie` response header
- **Automatically attached** to every subsequent request to that server
- **Scoped** — a cookie set by Server A is invisible to Server B
- Can be marked **HttpOnly** (no JavaScript access) and **Secure** (HTTPS only)

**Role in auth:** Server sets a cookie containing the session ID or JWT → browser sends it automatically with every request → no manual token management needed.

---

## Types of Authentication

### 1. Stateful Authentication (Session-Based)

```
Client                          Server                         Redis
  │                               │                              │
  │── username + password ──────→ │                              │
  │                               │── generate session ID ──────→│ store session ID + user data
  │← cookie (session ID, HttpOnly)│                              │
  │                               │                              │
  │── request + cookie ─────────→ │── lookup session ID ────────→│ return user data
  │← response ──────────────────  │                              │
```

**Pros:**
- Centralized control over all sessions
- Real-time info on active sessions
- Easy to revoke access (delete the session)
- Well-suited for most web apps with moderate traffic

**Cons:**
- Scalability issues with distributed systems
- Higher operational complexity (session store management, replication)

### 2. Stateless Authentication (JWT-Based)

```
Client                          Server
  │                               │
  │── username + password ──────→ │
  │                               │── generate signed JWT (secret key)
  │← JWT ────────────────────────│
  │                               │
  │── request + Authorization: Bearer <JWT> ──→ │
  │                               │── verify JWT with secret key
  │← response ──────────────────  │   (extract user ID, role from payload)
```

**Pros:**
- No session store needed — scales effortlessly
- Works across microservices (shared secret key)
- Mobile-friendly (no cookie dependency)

**Cons:**
- Token revocation is complex
- Token theft = full impersonation until expiry

**Hybrid approach:** Use stateful auth for web apps (browser clients), stateless auth for mobile apps and server-to-server communication.

### 3. API Key Authentication

For **machine-to-machine** (M2M) communication — no human interaction, no login form.

**How it works:**
1. User generates an API key from the platform's UI
2. Key is a cryptographically random string with specific permissions and expiry
3. Client sends the API key with each request (in headers)
4. Server identifies the caller, checks quotas/permissions

**Example:** OpenAI API — you generate a key, your server sends it to OpenAI's server to get ChatGPT responses programmatically.

**Pros:**
- Easy to generate and use
- Ideal for M2M / programmatic access
- Single-purpose, permission-scoped

**Use case:** Server-to-server communication, third-party API access, CLI tools

### 4. OAuth 2.0 + OpenID Connect (OIDC)

#### The Delegation Problem

One platform needs access to another platform's resources on behalf of a user:
- Travel app wants to scan your Gmail for flight tickets
- Social media app wants to import your Google contacts

Early solution: users **shared passwords** → full account access, no permission scoping, impossible to revoke without changing password everywhere. Disastrous.

#### OAuth 1.0 (2007) — The Revolution

Instead of sharing passwords, share **tokens** with specific, limited permissions.

**Four roles:**
| Role | Who | Example |
|------|-----|---------|
| Resource Owner | User who owns the data | You |
| Client | App requesting access | Facebook |
| Resource Server | Server hosting the resource | Google (contacts) |
| Authorization Server | Server that issues tokens | Google Auth |

**Flow:**
1. Client redirects user → authorization server
2. User authenticates + grants specific permissions
3. Auth server sends token → client
4. Client uses token → access resources on resource server

#### OAuth 2.0 (2010) — Simplified

Improvements over 1.0:
- **Bearer tokens** instead of complex cryptographic signatures
- Multiple **grant flows** for different app types:

| Flow | Use case |
|------|----------|
| **Authorization Code** | Server-side web apps |
| **Implicit** | Browser-based apps (now discouraged) |
| **Client Credentials** | Machine-to-machine (no user involved) |
| **Device Code** | Limited-input devices (Smart TVs) |

#### OpenID Connect (OIDC, ~2014) — Adding Authentication

OAuth 2.0 solved **authorization** (what can you access) but not **authentication** (who are you).

OIDC adds an **ID Token** (a JWT) containing:
- User ID (`sub`)
- When they logged in (`iat`)
- Issuing authority (`iss`)
- Name, email, profile picture

**This powers "Sign in with Google/Facebook/Discord"** — the platform takes your identity from Google and uses it for authentication without building its own auth system.

**OIDC Flow:**
1. Client redirects user → Google's auth server
2. User logs in + grants permissions
3. Auth server returns **authorization code** + **ID token**
4. Client exchanges auth code for **access token** + ID token
5. Client uses access token to access resources on behalf of user

---

## When to Use Which Authentication Type

| Type | Ideal for |
|------|-----------|
| **Stateful** (sessions) | Web apps, SaaS platforms — most common choice |
| **Stateless** (JWT) | APIs, distributed systems, mobile apps |
| **OAuth 2.0 / OIDC** | Third-party integrations, "Sign in with X" |
| **API Keys** | M2M communication, programmatic API access |

> In practice, you'll use stateful and stateless authentication most of the time.

---

## Authorization — Role-Based Access Control (RBAC)

Not all users should have the same capabilities. Example: a note-taking app where admins can access the "dead zone" (permanently deleted notes) but regular users cannot.

### How RBAC Works

Define **roles**, assign **permissions** to each role:

| Role | Permissions |
|------|------------|
| User | Read, Write, Delete (own notes) |
| Moderator | Read, Write (all notes) |
| Admin | Read, Write, Delete (all notes) + access dead zone |

### RBAC Flow

1. User registers → server assigns a **role** (user/admin/moderator)
2. User authenticates → sends token (session ID or JWT)
3. Server identifies user → deduces role (from token payload or DB lookup)
4. Role is attached to the request context early in the middleware chain
5. Downstream logic checks role → allows or denies the operation
6. Denied → `403 Forbidden` ("you don't have permission to access this resource")

You can go as **granular** as needed — different permissions per resource per role.

---

## Security Considerations

### 1. Generic Error Messages in Auth

**Never** send specific error messages during authentication:

| Bad (gives attackers clues) | Good (generic) |
|---|---|
| "User not found" → attacker knows username doesn't exist, tries next | "Authentication failed" |
| "Incorrect password" → attacker knows username is correct, brute-forces password | "Authentication failed" |
| "Account locked" → attacker knows the account exists | "Authentication failed" |

Always return the same generic message regardless of failure reason.

### 2. Timing Attacks

In a typical auth flow:
1. Find user by email → if not found, respond immediately (fast)
2. Check if account is locked
3. Hash provided password + compare with stored hash → if mismatch, respond (slower — hashing takes time)

**The problem:** If username is invalid, response is ~fast. If password is invalid, response is ~slower (due to hashing). Attackers measure this difference to determine which field failed.

**Defenses:**
- **Constant-time comparison functions** — execution time doesn't vary based on input similarity
- **Simulated delays** — even when username fails at step 1, add an artificial delay (e.g. `setTimeout` / `time.Sleep`) to equalize response times across all failure paths
