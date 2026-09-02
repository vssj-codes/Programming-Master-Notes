# Table-of-Contents

<!-- toc -->

- [Configuration Management](#configuration-management)
  * [What is Configuration Management?](#what-is-configuration-management)
  * [Why It Matters](#why-it-matters)
  * [Types of Configuration](#types-of-configuration)
    + [1. Application Settings](#1-application-settings)
    + [2. Database Configuration](#2-database-configuration)
    + [3. External Services](#3-external-services)
    + [4. Feature Flags](#4-feature-flags)
    + [5. Other Configuration Types](#5-other-configuration-types)
  * [Configuration Storage Options](#configuration-storage-options)
    + [Environment Variables](#environment-variables)
    + [Files](#files)
    + [Key-Value Stores](#key-value-stores)
    + [Cloud Secrets Management](#cloud-secrets-management)
    + [Hybrid Strategy (Common in Practice)](#hybrid-strategy-common-in-practice)
  * [Environment-Specific Configuration](#environment-specific-configuration)
  * [Security Best Practices](#security-best-practices)
    + [1. Never Hardcode Secrets](#1-never-hardcode-secrets)
    + [2. Use Cloud Secrets Management](#2-use-cloud-secrets-management)
    + [3. Access Control (Least Privilege)](#3-access-control-least-privilege)
    + [4. Rotation](#4-rotation)
    + [5. Validation at Startup ← Most Important](#5-validation-at-startup-%E2%86%90-most-important)
  * [Summary](#summary)

<!-- tocstop -->

---

# Configuration Management

**Source:** Sriniously — Backend from First Principles (Video 17)
**Link:** [Watch](https://www.youtube.com/watch?v=estH64OkwxU)

---

## What is Configuration Management?

**Configuration management** = the systematic approach to organizing, storing, accessing, and maintaining all the settings of your backend application.

Think of it as the **DNA of your application** — it determines how your code runs across different environments.

Most people think config management = storing secrets (DB passwords, API keys). That's like saying a car is just about its engine. Config management covers:

- How the application **starts up**
- How it **connects** to external services
- How it **behaves** across environments
- What it **logs** and where
- Where it sends **metrics**
- Which **features** are enabled and for which users

---

## Why It Matters

Modern backends run as part of distributed systems — multiple services, databases, caches, queues, third-party integrations. Each connection point requires configuration.

**Without a systematic approach → Configuration chaos:**
- Hardcoded values scattered throughout the codebase
- Inconsistent behavior across environments
- Security vulnerabilities from exposed secrets
- Impossible to reproduce issues because you can't trace which config caused a production break

**Misconfigured frontend:** shows a wrong dialogue, wrong redirect.
**Misconfigured backend:** exposes customer data, processes payments incorrectly, brings down the entire platform.

---

## Types of Configuration

### 1. Application Settings

The most common. Controls core server behavior.

| Setting | Example values |
|---------|---------------|
| Port | `8080` (local), varies in cloud |
| Log level | `debug` (dev), `info` (prod) |
| Connection pool size | `10` (local), `50` (prod) |
| Request timeout | e.g. `60s` — requests exceeding this get dropped with `504` |

### 2. Database Configuration

All details needed to connect to your database:
- Host, port, username, password, database name
- Combined into a connection URL
- Query timeout (how long a DB query runs before being killed)

### 3. External Services

API keys and connection details for any third-party service:
- Email providers (Resend, Mailchimp, Mailgun)
- Payment processors (Stripe)
- Authentication providers (Clerk, Auth0)
- Object storage (S3)

### 4. Feature Flags

Dynamically enable or disable features without deploying code.

**Example use case:** New checkout flow is built. Instead of rolling it out to all users:
- Enable it only for users from a specific region (A/B testing)
- Gradually roll out to more users
- Instantly roll back by toggling the flag off

Feature flags allow conditional feature access **at runtime** without code changes.

### 5. Other Configuration Types

| Type | Examples |
|------|---------|
| **Infrastructure / DevOps** | Kubernetes settings, cloud provider configs |
| **Security** | JWT secret, session secret, encryption keys |
| **Performance tuning** | Max CPU count (Go), thread pool sizes |
| **Business rules** | Maximum order amount, discount limits, rate thresholds |

---

## Configuration Storage Options

### Environment Variables

The most common approach. Supported in every language.

- Locally: `.env` file + a library (e.g. `dotenv` for Node.js) loads values into the OS environment
- In cloud/Kubernetes: the deployment pipeline fetches secrets from a secrets manager and injects them as env vars before the app starts

**Flow:**
```
Deploy triggered
  → Fetch configs from secrets manager (Vault / Parameter Store)
  → Inject as environment variables
  → App starts, reads from process.env / os.Getenv / etc.
```

### Files

| Format | Notes |
|--------|-------|
| **YAML** | Most common — supports comments, hierarchical structure |
| **JSON** | No comment support — less preferred for config |
| **TOML** | Growing adoption, clean syntax |

YAML config files are common in open-source projects — a single `config.yaml` centralizes all application settings with inline documentation.

### Key-Value Stores

Tools like **Consul** or **etcd** — lightweight, cloud-native. Works similarly to environment variables but with dynamic update support and service discovery.

### Cloud Secrets Management

Dedicated services for production-grade config management:

| Provider | Service |
|----------|---------|
| HashiCorp | **Vault** |
| AWS | **Parameter Store** / **Secrets Manager** |
| Azure | **Key Vault** |
| Google Cloud | **Secret Manager** |

These handle:
- **Encryption at rest** — secrets are stored encrypted
- **Encryption in transit** — encrypted when fetched
- **Access control** — fine-grained permissions
- **Audit logging** — who accessed what and when

### Hybrid Strategy (Common in Practice)

Merge configs from multiple sources with defined priority:

```
Priority 1: Cloud secrets manager (AWS Parameter Store)
Priority 2: config.yaml file
Priority 3: Environment variables
Priority 4: Application defaults
```

Higher priority sources override lower ones. Environment-specific behavior is controlled by which source is active.

---

## Environment-Specific Configuration

Each environment has different priorities → different config values.

| Environment | Priority | Example: DB pool size |
|-------------|---------|----------------------|
| **Development** (local) | Developer productivity, fast debugging | `10` |
| **Test** (CI) | Automated validation, quality assurance | `5` |
| **Staging** | Mirror production behavior, minimize cloud cost | `2` |
| **Production** | Reliability, security, performance | `50` |

The **application code stays the same** across environments — only the config changes. This is the key benefit of centralized config management: change behavior without touching code.

---

## Security Best Practices

### 1. Never Hardcode Secrets

Database URLs, API keys, JWT secrets — never commit these to your codebase. Use environment variables or a secrets manager.

### 2. Use Cloud Secrets Management

Over-engineer your secrets storage. Tools like HashiCorp Vault and AWS Parameter Store handle encryption at rest and in transit automatically. Worth the setup cost.

### 3. Access Control (Least Privilege)

Define who can access which configs:

| Role | Access |
|------|--------|
| Frontend engineers | Frontend API URLs, frontend integration keys |
| Backend engineers | DB credentials, Redis, Elasticsearch, backend service keys |
| DevOps | Cloud instance access, infrastructure configs |

No developer should have access to configs they don't need.

### 4. Rotation

Periodically rotate all secrets — API keys, JWT secrets, DB passwords. Reduces impact if a secret is leaked.

### 5. Validation at Startup ← Most Important

**Always validate your configuration before your server starts.**

Load all configs → validate with a schema validation library (Zod for TypeScript, Go Validator for Go) → if anything is missing or malformed, **fail immediately with a clear error message** before the app serves any traffic.

**Why this matters:**
- Without validation, a missing required variable may only cause a crash when a specific code path is hit by a real user
- With validation, the app fails at startup → CI/CD catches it → previous deployment keeps running (blue-green)

> This is the single most valuable practice in config management. A missing env var that silently breaks production is one of the hardest bugs to diagnose.

---

## Summary

| Aspect | Key point |
|--------|----------|
| **Scope** | Not just secrets — app settings, feature flags, performance tuning, business rules |
| **Types** | Application, database, external services, feature flags, security, infra, business rules |
| **Storage** | Env vars (most common), YAML files, key-value stores, cloud secrets managers |
| **Environments** | Different configs per env — same code, different behavior |
| **Security** | Never hardcode, use cloud managers, least privilege, rotate secrets |
| **Validation** | Always validate all configs at startup — fail fast before serving traffic |
