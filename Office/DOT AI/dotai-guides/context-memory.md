# DotAI Backend — Context Memory

> Use this file at the start of a new chat to restore full context instantly.
> Paste this file's contents and say: "Here is my context, continue from where we left off."

---

## User Preferences

- All `.md` guide/documentation files must be saved to:
  `/Users/vsrisaijyothi/Desktop/Programming-Master-Notes/DOT AI/dotai-guides/`
- Working directory: `/Users/vsrisaijyothi/Desktop/dotai-backend`

---

## Project Overview

- **Name:** DotAI Backend
- **Purpose:** Enterprise IoT asset tracking platform — tracks physical assets (vehicles, trailers, toolboxes, tags, gateways) across facilities in real-time
- **Stack:** NestJS 9.4, TypeScript, MongoDB (Mongoose), Redis, Keycloak, AWS (S3/SES/SNS/IoT Core), Socket.io, MQTT, Kubernetes
- **Entry point:** `src/main.ts` (port 5000)
- **Root module:** `src/app.module.ts`
- **60+ feature modules** in `src/models/`
- All Mongoose schemas → `submodules/entities-submodule/`
- All DTOs → `submodules/dtos-submodule/`
- Git remote: `https://github.com/vssj-codes/nestjs-backend.git` (branch: `main`)

---

## Architecture

### Multi-Tenancy
- Every HTTP request carries a `tenantId` (extracted from JWT by `TenantsMiddleware`)
- `TenantConnectionProvider` (REQUEST scoped) switches Mongoose to `tenant_{tenantId}` database
- All models are provisioned per-request via `createTenantModelProvider()`
- Complete data isolation between tenants at the database level

### Auth & Security
- **Keycloak** handles login — backend never sees raw passwords
- Every request → `KeycloakMiddleware` validates JWT (RS256) using Keycloak's JWKS public key
- `RolesGuard` reads `selected_tenant` claim from JWT → checks user's tenant roles vs endpoint roles
- All mutating endpoints (POST/PATCH/DELETE) protected by `CSRFGuard` (`csrf-token` header required)
- Standard roles: `admin`, `manager`, `operator`, `installer`, `viewer`, `groupadmin`, `developer`

### Data Patterns
- **Soft deletes:** `deletedAt` timestamp set first, then hard delete — never lose audit trail
- **Audit trail:** `createdBy`, `updatedBy`, `deletedBy` on every entity
- **Standard response:** `{ success: boolean, data: any, message: string }`
- **Pagination (v1):** `page`, `pageSize`, `searchText`, `sortBy`, `sortFrom` + returns `metaData`

### Real-Time
- Socket.io WebSocket gateway — clients join tenant-specific rooms on connect
- IoT data flow: Device → MQTT → AWS IoT Core → AWSMQTTModule → Socket.io → Client

---

## Key Files

| File | Purpose |
|---|---|
| `src/main.ts` | App bootstrap, global middleware, port 5000 |
| `src/app.module.ts` | Root module, all feature modules imported here |
| `src/middlewares/keycloak.middleware.ts` | JWT validation on every request |
| `src/middlewares/tenants.middleware.ts` | Tenant extraction and DB switching trigger |
| `src/guards/roles.guard.ts` | Per-endpoint role enforcement |
| `src/guards/csrf.guard.ts` | CSRF token validation on mutations |
| `src/auth/auth.service.ts` | JWT verify, token refresh, Keycloak admin API |
| `src/auth/auth.controller.ts` | `GET /auth/refresh-token` endpoint |
| `src/helpers/auth.helpers.ts` | JWT decode helpers, admin token fetcher |
| `src/models/vehicles/` | Reference module — most complete example |
| `submodules/helpers-submodule/provider/tenant-connection.provider.ts` | DB switching per request |
| `submodules/helpers-submodule/provider/tenant-model.provider.ts` | Model factory per tenant |
| `src/utils/socket/socket.gateway.ts` | WebSocket real-time gateway |
| `src/models/common/common.service.ts` | Shared utility methods across modules |

---

## NestJS Learning Progress

We went through NestJS from scratch. Completed topics:

### Stage 1 — Core Fundamentals (DONE)
| Step | Topic | Status |
|---|---|---|
| 1 | What is NestJS | Done |
| 2 | Modules | Done |
| 3 | Controllers | Done |
| 4 | Services | Done |
| 5 | Dependency Injection | Done |

### Stage 2 — Request Lifecycle (DONE)
| Step | Topic | Status |
|---|---|---|
| 6 | Middleware | Done |
| 7 | Guards | Done |
| 8 | Pipes | Done |
| 9 | Interceptors | Done |
| 10 | Exception Filters | Done |

### Stage 3 — Data & Config (NEXT)
| Step | Topic | Status |
|---|---|---|
| 11 | DTOs & Validation | **Next up** |
| 12 | Mongoose / MongoDB integration | Pending |
| 13 | Environment Config | Pending |

### Stage 4 — Advanced (Pending)
| Step | Topic | Status |
|---|---|---|
| 14 | Multi-tenancy | Pending |
| 15 | WebSockets | Pending |
| 16 | Job Queues (Bull) | Pending |
| 17 | MQTT | Pending |

---

## Guide Files Already Generated

| File | Contents |
|---|---|
| `KT-dotai-backend.md` | Full Knowledge Transfer of the entire codebase |
| `nestjs-fundamentals.md` | NestJS Steps 1–5 with real code from this codebase |
| `nestjs-stage2-request-lifecycle.md` | NestJS Steps 6–10 — full request lifecycle |
| `library-tracker-checklist.md` | Step-by-step build checklist for library-tracker project |
| `context-memory.md` | This file — session context for new chats |

---

## How to Continue in a New Chat

1. Open a new Claude Code chat in `/Users/vsrisaijyothi/Desktop/dotai-backend`
2. Say: *"Read my context file at `/Users/vsrisaijyothi/Desktop/Programming-Master-Notes/DOT AI/dotai-guides/context-memory.md` and continue my NestJS learning from Step 11: DTOs & Validation"*
3. Claude will have full context and continue from exactly where we left off

---

*Last updated: 2026-03-23*
