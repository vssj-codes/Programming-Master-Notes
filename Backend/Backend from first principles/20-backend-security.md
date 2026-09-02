# Table-of-Contents

<!-- toc -->

- [Backend Security](#backend-security)
  * [The Security Mindset](#the-security-mindset)
  * [The Multi-Language Model — Root Cause of Injection Attacks](#the-multi-language-model--root-cause-of-injection-attacks)
  * [Injection Attacks](#injection-attacks)
    + [SQL Injection](#sql-injection)
    + [NoSQL Injection](#nosql-injection)
    + [Command Injection](#command-injection)
    + [Injection Attack Mental Model](#injection-attack-mental-model)
  * [Authentication Security](#authentication-security)
    + [Password Storage](#password-storage)
    + [Sessions (Stateful Authentication)](#sessions-stateful-authentication)
    + [JWT (Stateless Authentication)](#jwt-stateless-authentication)
    + [Rate Limiting for Authentication](#rate-limiting-for-authentication)
  * [Authorization Security](#authorization-security)
    + [BOLA — Broken Object Level Authorization](#bola--broken-object-level-authorization)
    + [BFLA — Broken Function Level Authorization](#bfla--broken-function-level-authorization)
    + [Horizontal vs Vertical Authorization Attacks](#horizontal-vs-vertical-authorization-attacks)
    + [Authorization Best Practices](#authorization-best-practices)
  * [XSS — Cross-Site Scripting](#xss--cross-site-scripting)
    + [Types of XSS](#types-of-xss)
    + [Root Cause](#root-cause)
    + [Prevention](#prevention)
  * [CSRF — Cross-Site Request Forgery](#csrf--cross-site-request-forgery)
  * [Misconfigurations](#misconfigurations)
    + [Secrets in Version Control](#secrets-in-version-control)
    + [Debug Mode in Production](#debug-mode-in-production)
    + [Missing Security Headers](#missing-security-headers)
  * [The Boundary Mental Model](#the-boundary-mental-model)
  * [Defense in Depth](#defense-in-depth)
  * [Auth Provider Recommendation](#auth-provider-recommendation)
  * [Resources](#resources)

<!-- tocstop -->

---

# Backend Security

**Source:** Sriniously — Backend from First Principles (Video 20)
**Link:** [Watch](https://www.youtube.com/watch?v=estH64OkwxU)

---

## The Security Mindset

Security is not a feature you add — it is a **mindset** you bring to every line of code. No application can ever be completely secure; as technology evolves, new vulnerabilities emerge. The goal is continuous improvement.

Attackers don't care about your framework, language, or library. They ask one question:

> **Where did the developer make an assumption?**

Common dangerous assumptions:
- "The input from the user will be clean"
- "The user is who they say they are"
- "The request is coming from my own front end"
- "No one will open the network tab and modify parameters"

These feel reasonable under deadline pressure when you're thinking in the happy path. Attackers don't use your app the way you intend — they poke at every boundary and break every assumption.

**The discipline:** Every time you write code, ask _"What could go wrong here in terms of security?"_

---

## The Multi-Language Model — Root Cause of Injection Attacks

Your backend speaks multiple languages simultaneously:

| Context | Language |
|---------|----------|
| Database queries | SQL |
| Rendered content | HTML / CSS / JavaScript |
| OS operations | Shell / system calls |

Each language has its own grammar, special characters, and ways of separating **commands from data**.

Most vulnerabilities arise when **user input in one language crosses the boundary into another language**. The user's data gets interpreted as code in the target language — that is the root cause of injection attacks.

```
User (browser/HTML) → crosses boundary → SQL (database)
                                       → Shell (OS)
                                       → HTML (browser DOM)
```

---

## Injection Attacks

### SQL Injection

When a server constructs a SQL query by concatenating user input directly into the query string, the user's input can contain SQL syntax that alters the query's meaning.

**Vulnerable pattern:**
```sql
SELECT * FROM users WHERE email = '<user_input>'
```

**Malicious input:** `' OR '1'='1' --`

**Resulting query:**
```sql
SELECT * FROM users WHERE email = '' OR '1'='1' --'
```

What happened:
- The opening `'` closes the server's quote, completing `email = ''`
- `OR '1'='1'` is always true → the WHERE clause is bypassed
- `--` comments out the trailing quote, preventing a syntax error
- Result: all rows returned

Attackers can also inject `DROP TABLE`, `UNION SELECT` to read other tables, or database functions to read files from the server's filesystem.

**Fix — Parameterized Queries (Prepared Statements):**

```go
// WRONG: string concatenation
query := "SELECT * FROM users WHERE email = '" + userInput + "'"

// CORRECT: parameterized query
statement := "SELECT * FROM users WHERE email = $1"
db.Query(statement, userInput)
```

The key principle: **send the query template and the data separately**. The database treats the parameter slot as pure data — no SQL syntax inside it is ever interpreted as a command.

> Every modern database driver and ORM uses parameterized queries by default. The only way to be vulnerable is to deliberately bypass this by building raw strings yourself.

### NoSQL Injection

MongoDB queries are JSON-like objects. If an application passes user-controlled JSON directly to a query, attackers can inject MongoDB operators (e.g., `$ne`, `$gt`, `$exists`) to manipulate query logic.

**Fix:** Never pass raw user-controlled objects to database queries. Validate input structure strictly.

### Command Injection

When a backend calls OS commands or CLI tools (e.g., FFmpeg for image processing) by concatenating user input into the command string:

**Vulnerable pattern:**
```
ffmpeg -h 120 -w 220 -o <user_filename>
```

**Malicious input:** `output.jpg; rm -rf /`

**Resulting command:**
```
ffmpeg -h 120 -w 220 -o output.jpg; rm -rf /
```

The semicolon ends the ffmpeg command and starts a new destructive one.

**Fix:** Use language-provided functions that accept the command and arguments as separate parameters. The arguments are passed directly to the process without going through a shell interpreter — never treated as code.

### Injection Attack Mental Model

| Do | Don't |
|----|-------|
| Use parameterized queries for DB | Concatenate user input into SQL strings |
| Use argument arrays for OS calls | Build shell commands by string concatenation |
| Validate input structure at the boundary | Assume user input is safe |

> **Rule:** Whenever you're building a string that will be interpreted by another system and that string includes user input — stop. Find the parameterized alternative. It almost always exists.

---

## Authentication Security

### Password Storage

**Never store passwords in plain text.** Database breaches happen constantly. Exposed plain text passwords compromise users across all sites (70%+ of users reuse passwords).

**Evolution of password storage:**

| Approach | Problem |
|----------|---------|
| **Plain text** | Breach exposes all passwords directly |
| **Basic hashing (MD5, SHA-256)** | Rainbow table attacks — precomputed hashes of common passwords can be reverse-matched |
| **Hashing + salting** | A random salt unique per user is appended before hashing — same password produces different hashes for each user, defeating rainbow tables |
| **Slow hash functions (bcrypt, Argon2ID)** | Deliberately slow; makes offline brute-force take decades instead of days |

**How salting works:**
1. Generate a cryptographically random salt for each user
2. Hash `password + salt` using a slow hash function
3. Store both the hash and the salt in the database
4. On login: retrieve salt, hash the provided password with it, compare

**Why slow hash functions matter:**
- A GPU can compute billions of SHA-256 hashes/second
- bcrypt/Argon2ID with a cost factor of 400ms: 4–5 attempts/second instead of billions
- Brute forcing goes from days to centuries

**Current industry standard:** Argon2ID (successor to bcrypt). Use the cost factor your system can afford — 100–400ms is imperceptible to a real user but devastating to an attacker.

---

### Sessions (Stateful Authentication)

After successful login, the server:

1. **Generates a session ID** — 128–256 bit cryptographically secure random string
2. **Stores the session** in a database (Redis or primary DB) with metadata: user ID, creation time, expiry, IP address, user agent
3. **Sends the session ID** to the client as a cookie

For each subsequent request, the server extracts the session ID from the cookie, looks it up in the store, and identifies the user.

**Cookie Security Flags:**

| Flag | Value | Effect |
|------|-------|--------|
| `HttpOnly` | `true` | JavaScript cannot read the cookie — prevents XSS from stealing the session ID |
| `Secure` | `true` | Cookie only sent over HTTPS — prevents interception on public networks |
| `SameSite` | `Strict` or `Lax` | Controls cross-origin sending — primary defense against CSRF attacks |

> Never store session IDs or JWTs in `localStorage` — it is accessible to any JavaScript, including injected malicious scripts.

---

### JWT (Stateless Authentication)

A JWT has three parts:

```
header.payload.signature
```

| Part | Content |
|------|---------|
| **Header** | Algorithm used (e.g., HS256) |
| **Payload** | Claims: `sub` (user ID), `iat` (issued at), custom fields (name, role) |
| **Signature** | HMAC of header + payload using a server-side secret |

If an attacker tampers with the payload, the signature becomes invalid and the server rejects the token.

> The payload is only Base64-encoded, not encrypted. Never store sensitive data in JWT claims — anyone can decode and read it.

**JWT Weaknesses:**

| Problem | Workaround |
|---------|-----------|
| **Revocation is hard** — can't instantly invalidate a token | Blacklist tokens in Redis; use short expiry + refresh token flow |
| **Storage risk** — localStorage vulnerable to XSS | Use `HttpOnly` cookies instead |
| **Scaling assumption** — trades DB roundtrips for complexity | Fine for horizontal scaling, but adds complexity |

**Refresh Token Flow:**
- Issue an **access token** (5–15 minute expiry) and a **refresh token** (1–7 day expiry)
- Client uses access token for API calls; when it expires (401), client exchanges refresh token for a new pair
- If compromised: access token expires quickly; refresh token expiry limits damage window

**Recommendation:** Unless you have specific horizontal scaling needs, prefer stateful session-based authentication. Revocation is simpler and the architecture is more straightforward. If using JWTs, always use short expiry, refresh token flows, and `HttpOnly` cookies.

---

### Rate Limiting for Authentication

Without rate limiting, attackers can attempt millions of password combinations or flood your server until it crashes.

**Layered rate limiting strategy:**

| Layer | Mechanism | Limits |
|-------|-----------|--------|
| **Per-IP** | Limit login attempts per IP per time window | e.g., 10 attempts/minute per IP |
| **Per-account** | Lock account after N failed attempts | e.g., 5 failures/15 min → lock for 24 hours |
| **Global** | System-wide login attempt ceiling | e.g., 100 failed attempts/minute triggers alert |

Each layer addresses what the others miss: IP rotation defeats per-IP; account targeting defeats per-account; global limits catch distributed attacks that bypass both. Be more restrictive on auth endpoints than on general API endpoints.

---

## Authorization Security

Authorization answers: **What is this authenticated user allowed to do?**

### BOLA — Broken Object Level Authorization

Also called **Insecure Direct Object Reference (IDOR)**.

**The mistake:** After authenticating the user and checking they have "read invoices" permission at the routing layer, the repository query does not scope results to the user's own data.

```sql
-- WRONG: fetches any invoice regardless of owner
SELECT * FROM invoices WHERE id = 5

-- CORRECT: scopes to the authenticated user
SELECT * FROM invoices WHERE id = 5 AND user_id = <context.userId>
```

**Important:** When an unauthorized access attempt is made, return `404 Not Found` — not `403 Forbidden`. A 403 confirms the resource exists, enabling attackers to enumerate valid IDs.

**Applies to all operations:** SELECT, INSERT, UPDATE, DELETE — any database interaction must verify ownership at the point of access.

### BFLA — Broken Function Level Authorization

Restricting access to **admin-level functions**, not just data.

**The mistake:** Assuming that keeping an admin URL secret is security ("security through obscurity"). Any user who discovers the URL can call admin endpoints without a role check.

**Fix:** Add a role check middleware at the routing layer in addition to the permission check:
```
require_auth → check permission (read:invoices) → check role (admin) → handler
```

### Horizontal vs Vertical Authorization Attacks

| Attack type | Description | Vulnerability |
|-------------|-------------|---------------|
| **Horizontal** | User A accesses User B's resources | BOLA / IDOR |
| **Vertical** | Regular user accesses admin-level functions | BFLA |

### Authorization Best Practices

1. **Centralize authorization logic** — a single, consistent layer; not scattered across the codebase
2. **Default deny** — if not explicitly permitted, block it. New endpoints are protected until explicitly granted
3. **Test authorization specifically** — automated tests for: user A cannot access user B's data; regular users cannot call admin functions; unauthenticated requests are rejected
4. **Audit logs** — log every sensitive access (admin endpoints, failed authorization checks) for forensics and alerting
5. **Use UUIDs for IDs** — sequential integer IDs are predictable and enable enumeration attacks; random UUIDs eliminate this

---

## XSS — Cross-Site Scripting

XSS occurs when an attacker gets their JavaScript to execute **in a victim's browser, in the context of your platform**.

**Why it's dangerous:** Malicious JS running in your site's context can:
- Read all page content including sensitive data
- Make API requests as the logged-in user
- Steal cookies (if not `HttpOnly`) and `localStorage`
- Redirect users to phishing pages
- Alter page content to deceive users

### Types of XSS

| Type | How it works |
|------|--------------|
| **Stored XSS** | Malicious script saved to DB (e.g., in a comment); served to all viewers |
| **Reflected XSS** | Malicious script embedded in a URL parameter; reflected back in the response |
| **DOM-based XSS** | Script injected via `innerHTML` / `dangerouslySetInnerHTML` from user-controlled input |

### Root Cause

Same as injection attacks: **user-provided data being treated as code instead of data** — but at the browser/HTML layer rather than the database layer.

### Prevention

1. **Sanitize user input** — strip or escape `<script>` tags and dangerous HTML before storing or rendering
2. **Content Security Policy (CSP)** — HTTP header from your server that tells the browser:
   - Only execute scripts from specific trusted domains
   - Block all inline scripts
   - Restrict image/resource sources

> CSP is a last line of defense, not a primary prevention. Fix the root cause (sanitization) first; CSP catches what slips through.

---

## CSRF — Cross-Site Request Forgery

A malicious site tricks a user's browser into making a request to your site, which the browser sends with the user's cookies — making the server think it's a legitimate request.

**Example:** User is logged into `bank.com`. They visit `evil.com` which auto-submits a form that transfers money to `bank.com`. The browser includes the `bank.com` cookie automatically.

**Modern status:** Not a major threat for modern applications because:

| Defense | How it helps |
|---------|-------------|
| **SameSite cookie** set to `Strict` or `Lax` | Browser won't send cookie on cross-site requests |
| **CORS configuration** | Server rejects requests not originating from your own front end |

Modern browsers default `SameSite` to `Lax`. As long as you're not setting it to `None`, you are largely protected by default.

---

## Misconfigurations

### Secrets in Version Control

Never commit API keys, database passwords, JWT secrets, or encryption keys to Git. Even if deleted, they remain in commit history.

**If you accidentally commit a secret:** Immediately rotate/revoke it and generate a new one.

**Use instead:** Environment variables, `.env` files (not committed), or cloud secrets managers (AWS Parameter Store, HashiCorp Vault, Azure Key Vault).

### Debug Mode in Production

| Log level | What gets logged |
|-----------|-----------------|
| `debug` (dev) | Detailed errors, stack traces, SQL queries, DB config, sensitive data |
| `info` (prod) | Business events only — no internal details |

If `debug` level is left on in production and logs are breached, attackers gain detailed knowledge of your code structure, database schema, and user data.

**Always explicitly configure log level per environment.**

### Missing Security Headers

| Header | Protection |
|--------|-----------|
| **Content-Security-Policy** | Restricts what resources/scripts the browser executes |
| **X-Frame-Options** | Prevents your site from being embedded in iframes (blocks clickjacking) |
| Others (HSTS, X-Content-Type-Options, etc.) | Various browser-level protections |

Most modern web frameworks provide a one-line security middleware that configures all standard security headers automatically. Use it.

---

## The Boundary Mental Model

> Every vulnerability we discussed happens when **data crosses a boundary**.

| Vulnerability | Boundary crossed |
|---------------|-----------------|
| SQL injection | User input → SQL query language |
| Command injection | User input → OS shell |
| XSS | User markup → HTML/JavaScript in browser |
| BOLA | Request scope → another user's data |
| BFLA | User role → admin-level function access |

**Ask yourself at every boundary:**
1. Where is data crossing a boundary?
2. What assumptions am I making about this data?
3. What if those assumptions are wrong?

---

## Defense in Depth

No single defense is perfect. Layer them:

```
1. Input validation         ← first line; strict schema at the entry point
2. Parameterized operations ← separate commands from data (DB, OS)
3. Authorization at point of access ← check ownership at the repo layer, not just routing
4. Security headers & CSP   ← limit damage if something slips through
5. Monitoring & alerting    ← detect suspicious activity in real time
```

An attacker must bypass all layers simultaneously — dramatically reducing the probability of a successful attack.

---

## Auth Provider Recommendation

For production systems, consider using a managed auth provider (Clerk, Auth0, etc.) instead of building your own:

**Benefits:**
- Handles stateful sessions, social OAuth (Google, GitHub), account linking, device management, and session revocation
- Dedicated security team responding to new attacks 24/7
- World-class UX out of the box with minimal integration effort

**When to self-implement:** When auth provider costs become prohibitive at scale (millions of users, $10k+/month), and you have the engineering bandwidth to do it properly.

Even with an auth provider, you must understand these concepts to configure it securely and integrate it correctly.

---

## Resources

| Resource | What it covers |
|----------|---------------|
| **[PortSwigger Academy](https://portswigger.net/web-security)** | Free, comprehensive labs: SQL injection, XSS, CSRF, SSRF, OAuth, JWT attacks, and more |
| **[OWASP Top 10](https://owasp.org/www-project-top-ten/)** | Current list of the most critical web application vulnerabilities with real-world instances |
| **[OWASP Cheat Sheets](https://cheatsheetseries.owasp.org/)** | Deep-dive best practices for authentication, session management, input validation, etc. |
| **[Lucia Auth](https://lucia-auth.com/)** (guidance) | Industry best practices for implementing secure authentication yourself |
