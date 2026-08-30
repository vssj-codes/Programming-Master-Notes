# DotAI Backend — Knowledge Transfer (KT)

> **Project:** DotAI Backend — Enterprise IoT Asset Tracking Platform
> **Stack:** NestJS 9.4, TypeScript, MongoDB, Redis, Keycloak, AWS, Socket.io, MQTT, Kubernetes
> **Date:** 2026-03-21

---

## Table of Contents

1. [What This Project Is](#1-what-this-project-is)
2. [Tech Stack at a Glance](#2-tech-stack-at-a-glance)
3. [App Bootstrap](#3-app-bootstrap--srcmaints)
4. [Module Architecture](#4-module-architecture)
5. [Request Lifecycle](#5-request-lifecycle)
6. [Multi-Tenancy](#6-multi-tenancy--how-it-works)
7. [Auth & Security](#7-auth--security)
8. [Controllers — Route Patterns](#8-controllers--route-patterns)
9. [Services — DB Patterns](#9-services--db-patterns)
10. [Real-Time — WebSockets](#10-real-time--websockets)
11. [API Conventions](#11-api-conventions)
12. [Key NPM Scripts](#12-key-npm-scripts)
13. [Where to Start Reading Code](#13-where-to-start-reading-code)

---

## 1. What This Project Is

An **enterprise IoT asset tracking platform** — it tracks physical assets (vehicles, trailers, toolboxes, tags, gateways) across facilities in real-time. Multiple companies (tenants) use the same platform but with completely isolated data.

---

## 2. Tech Stack at a Glance

| Layer | Technology |
|---|---|
| Framework | NestJS 9.4, TypeScript, Node 20 |
| Database | MongoDB 7 via Mongoose |
| Auth | Keycloak (JWT, RS256) |
| Cache | Redis via ioredis |
| Real-time | Socket.io (WebSockets) |
| IoT Messaging | MQTT, AWS IoT Core |
| Cloud | AWS S3, SES, SNS, Secrets Manager, SSM |
| Infra | Docker, Kubernetes, Terraform |
| CI/CD | GitLab CI (11 pipeline configs) |
| Testing | Jest (unit), Cypress (e2e) |

---

## 3. App Bootstrap — `src/main.ts`

When the app starts, it:

1. Creates a NestJS app with **Winston** as the logger
2. Enables **URI-based API versioning** (`/v1/`, `/v2/`)
3. Applies global middleware:
   - `helmet()` — security headers + HSTS
   - `mongoSanitize()` — blocks NoSQL injection
   - `cookieParser()` — reads cookies (for refresh token)
   - `RequestResponseTimeMiddleware` — logs request timing
4. Enables **CORS** with a whitelist of allowed domains
5. Sets up **Swagger** docs
6. Listens on **port 5000**

---

## 4. Module Architecture

### Root Module — `src/app.module.ts`

The entry point. Imports all 60+ feature modules and applies two global middlewares:

| Middleware | Purpose | Applied To |
|---|---|---|
| `TenantsMiddleware` | Extracts tenantId from JWT, sets `req.tenantId` | All routes (with exclusions) |
| `KeycloakMiddleware` | Validates JWT signature, sets `req.roles` | All routes (with exclusions) |

**Excluded routes** (public/internal — no auth required):
- `auth/refresh-token`
- `misc/test-connection`, `misc/version`
- `logging/login`, `logging/add-selected-tenant`
- `aws-mqtt/*`, `camera/*`, `camera-detection/*`
- `socket/asset-tag-real-time-data`, `socket/chirp-real-time-data`

### Feature Modules (62 total in `src/models/`)

Each module follows this pattern:

```
vehicles/
  ├── vehicles.module.ts      ← wires everything together
  ├── vehicles.controller.ts  ← HTTP routes
  ├── vehicles.service.ts     ← business logic
  ├── dto/                    ← request/response shapes
  └── interface/              ← TypeScript interfaces
```

### Full Module List

| Domain | Modules |
|---|---|
| Asset Management | asset-tags, asset-tag-items, asset-tag-zones, asset-gateways, asset-bridge, asset-camera-histories |
| Tracking Entities | vehicles, trailers, toolboxes |
| Location | location, locate, locate-cache, heat-map, geofence |
| IoT Infrastructure | ap-cells, ap-wifis, booster-cells, extender-wifis, reader, barcode-scanner, camera, camera-detection |
| Tag Protocols | esl, esl-items, esl-oap, transit-tag, zim-tag, chirp, wiliot, wiliot-handler |
| Data Bridges | asset-bridge, marker-bridge, transit-bridges, access-bridge |
| Organization | structure, floorplan, zones, items |
| Users & Auth | users, tenant-users, keycloak, logging |
| System | settings, history, compute-node, kubernetes, releases |
| Infrastructure | aws-mqtt, ses-handler, sns-handler, cache-module, csrf-module, file |
| Utilities | common, misc, pricer, workorder |

---

## 5. Request Lifecycle

Every request passes through this pipeline:

```
HTTP Request
  │
  ├─ TenantsMiddleware      → extracts tenantId, switches DB
  ├─ KeycloakMiddleware     → validates JWT, sets roles
  │
  ├─ RolesGuard             → checks user's tenant roles vs endpoint roles
  ├─ CSRFGuard              → validates csrf-token header (mutations only)
  │
  ├─ Controller             → receives request, calls service
  ├─ Service                → business logic, DB queries
  │
  └─ HTTP Response          → { success, data, message }
```

---

## 6. Multi-Tenancy — How It Works

This is the most important architectural pattern in the codebase.

### The Flow

```
Request arrives with JWT
  → TenantsMiddleware decodes JWT → extracts tenantId
  → Sets req.tenantId = "abc123"
  → TenantConnectionProvider (REQUEST scoped) picks it up
  → Calls mongoose connection.useDb("tenant_abc123")
  → All models for this request query "tenant_abc123" database
  → Next request for a different tenant → different DB
```

### Key Files

| File | Role |
|---|---|
| `submodules/helpers-submodule/provider/tenant-connection.provider.ts` | Switches Mongoose DB connection per request |
| `submodules/helpers-submodule/provider/tenant-model.provider.ts` | Creates Mongoose models bound to the tenant DB |

### How Models Are Registered (in every module)

```typescript
// vehicles.module.ts
providers: [
  createTenantModelProvider(Vehicles.name, VehicleSchema, PROVIDER.VEHICLE_MODEL),
  createTenantModelProvider(Location.name, LocationSchema, PROVIDER.LOCATION_MODEL),
  // ...
]
```

### How Models Are Injected (in every service)

```typescript
// vehicles.service.ts
constructor(
  @Inject(PROVIDER.VEHICLE_MODEL)
  private vehicleModel: Model<VehiclesDocuments>,
)
```

> The model is already pointing to the correct tenant's DB by the time the service uses it.

### Database Naming Convention

```
tenant_{tenantId}   e.g.  tenant_abc123,  tenant_xyz789
```

---

## 7. Auth & Security

### JWT Flow (Login)

```
Client
  → Logs in via Keycloak (backend never sees passwords)
  → Receives JWT access token + refresh token
  → Sends "Authorization: Bearer <token>" on every request
  → KeycloakMiddleware validates JWT signature using Keycloak's public key (JWKS)
  → RolesGuard reads "selected_tenant" claim from JWT
  → Checks user's tenant roles against endpoint's required roles
  → Request proceeds or 403 Forbidden
```

### Key Auth Files

| File | Role |
|---|---|
| `src/middlewares/keycloak.middleware.ts` | Extracts & verifies JWT on every request |
| `src/auth/auth.service.ts` | JWT verification via RS256, token refresh, Keycloak admin API |
| `src/auth/auth.controller.ts` | `GET /auth/refresh-token` endpoint |
| `src/guards/roles.guard.ts` | Per-endpoint role enforcement |
| `src/guards/csrf.guard.ts` | CSRF token validation on mutations |
| `src/helpers/auth.helpers.ts` | JWT decode helpers + admin token fetcher |

### KeycloakMiddleware Steps (`src/middlewares/keycloak.middleware.ts`)

1. Extract Bearer token from `Authorization` header
2. Decode JWT **header** (not payload) to get `kid` (Key ID)
3. Fetch matching public key from Keycloak's JWKS endpoint
4. Verify JWT signature using RS256 + validate audience, issuer, maxAge
5. Attach roles to `req.roles`

### RolesGuard Steps (`src/guards/roles.guard.ts`)

1. Read required roles from `@Roles(...)` decorator on the endpoint
2. Decode JWT payload — extract `selected_tenant` attribute
3. `selected_tenant` contains the user's roles for their currently active tenant
4. Check if any user role matches required roles → allow or 403

### CSRF Guard (`src/guards/csrf.guard.ts`)

- All POST/PATCH/PUT/DELETE endpoints use `@UseGuards(CSRFGuard)`
- Client must send `csrf-token` header — a JWT signed with `JWT_SECRET`
- GET endpoints are exempt

### Token Refresh (`src/auth/auth.controller.ts`)

```
GET /auth/refresh-token
  → Reads refreshToken from httpOnly cookie
  → Calls Keycloak token endpoint with grant_type: refresh_token
  → Returns new accessToken in response body
  → Sets new refreshToken as httpOnly cookie (domain: .daic.ai)
```

### Standard Roles

`admin` | `manager` | `operator` | `installer` | `viewer` | `groupadmin` | `developer`

---

## 8. Controllers — Route Patterns

Using `VehiclesController` as the reference:

```typescript
@ApiTags('Vehicle')           // Swagger grouping
@ApiBearerAuth()              // Swagger auth annotation
@Controller('vehicles')       // base route
@Version('1')                 // /v1/vehicles
export class VehiclesController {

  // GET /v1/vehicles  — paginated list
  @Get()
  @UseGuards(RolesGuard)
  @Roles('admin', 'manager', 'viewer', 'installer', 'groupadmin', 'developer')
  findAll(@Query() query, @Req() req) { ... }

  // GET /v1/vehicles/:id
  @Get('/:id')
  @UseGuards(RolesGuard)
  @Roles('admin', 'manager', 'viewer', ...)
  findOne(@Param('id') id: string) { ... }

  // POST /v1/vehicles  — create
  @Post()
  @UseGuards(CSRFGuard, RolesGuard)
  @Roles('admin', 'installer')
  create(@Body() dto: CreateVehicleDto) { ... }

  // PATCH /v1/vehicles/:id  — update
  @Patch('/:id')
  @UseGuards(CSRFGuard, RolesGuard)
  @Roles('admin', 'installer')
  update(@Param('id') id: string, @Body() dto) { ... }

  // DELETE /v1/vehicles/:id  — soft delete
  @Delete('/:id')
  @UseGuards(CSRFGuard, RolesGuard)
  @Roles('admin', 'installer')
  delete(@Param('id') id: string) { ... }
}
```

### Pagination (v1 endpoints)

```
GET /v1/vehicles?page=1&pageSize=20&searchText=ford&sortBy=name&sortFrom=asc
```

---

## 9. Services — DB Patterns

Four main MongoDB access patterns used across all services:

```typescript
// 1. Simple find (read-only, fast)
this.vehicleModel.find({ deletedAt: null }).lean()

// 2. Aggregation (joins, complex transformations)
this.vehicleModel.aggregate([
  { $match: { deletedAt: null } },
  { $lookup: { from: 'locations', localField: '_id', foreignField: 'entityId', as: 'location' } },
  { $unwind: { path: '$location', preserveNullAndEmptyArrays: true } },
])

// 3. Update and return new document
this.vehicleModel.findOneAndUpdate(
  { _id: id },
  { $set: { name: 'new name', updatedAt: Date.now() } },
  { new: true }
)

// 4. Soft delete
this.vehicleModel.findOneAndUpdate(
  { _id: id },
  { $set: { deletedAt: Date.now(), deletedBy: userId } }
)
```

### Soft Delete Pattern

- Entities are **never hard deleted** from the DB immediately
- Instead `deletedAt` is set to current timestamp
- All queries filter: `{ deletedAt: null }`
- Hard delete follows soft delete only in certain flows

### Audit Trail Fields

Every entity has: `createdBy`, `updatedBy`, `deletedBy`, `createdAt`, `updatedAt`, `deletedAt`

### Standard Response Format

```typescript
// Success
{ success: true, data: {...}, message: 'Vehicle fetched successfully' }

// Error
{ success: false, data: null, message: 'Something went wrong' }
```

---

## 10. Real-Time — WebSockets

**File:** `src/utils/socket/socket.gateway.ts`

### Architecture

```
IoT Device
  → MQTT
  → AWS IoT Core
  → AWSMQTTModule (NestJS)
  → EventEmitter
  → Socket Gateway
  → Client (browser/app) via WebSocket
```

### Tenant Isolation

- On connect, every client joins a **tenant-specific room**: `client.join(tenantId)`
- All events are broadcast to tenant rooms: `wss.to(tenantId).emit(event, payload)`
- A user from Tenant A **never** receives data from Tenant B

### Key Events

| Event | Direction | Purpose |
|---|---|---|
| `Asset_Tag_Real_Time_Data` | Server → Client | Live asset tag location updates |
| `Receive_Message` | Server → Client | General messages |
| `getLocateData` | Client → Server | Request current location of entities |

### Entity Types Supported for Real-Time Locate

Structures, vehicles, trailers, toolboxes, asset gateways, AP cells, AP WiFis, booster cells, extender WiFis, ESL tags, transit gateways, Zim tags

---

## 11. API Conventions

| Convention | Detail |
|---|---|
| Base URL | `http://localhost:5000` |
| Versioning | `/v1/vehicles`, `/v2/vehicles` |
| Auth header | `Authorization: Bearer <jwt>` |
| CSRF header | `csrf-token: <signed-jwt>` (mutations only) |
| Response shape | `{ success: boolean, data: any, message: string }` |
| Pagination params | `page`, `pageSize`, `searchText`, `sortBy`, `sortFrom` |
| Soft deletes | `deletedAt` timestamp — never truly deleted |
| Audit fields | `createdBy`, `updatedBy`, `deletedBy` on every entity |
| Schemas location | `submodules/entities-submodule/` |
| DTOs location | `submodules/dtos-submodule/` |
| Swagger docs | `http://localhost:5000/api` |

---

## 12. Key NPM Scripts

```bash
npm run start:dev     # development with hot reload
npm run build         # compile TypeScript → dist/
npm run start:prod    # run compiled app (node dist/main)
npm run test          # Jest unit tests
npm run test:cov      # Jest with coverage report
npm run cy:open       # Cypress e2e tests (interactive)
npm run cy:run        # Cypress e2e tests (headless Chrome)
npm run lint          # ESLint with auto-fix
npm run format        # Prettier formatting
```

---

## 13. Where to Start Reading Code

Read in this order to build your mental model:

| Step | File | What You'll Learn |
|---|---|---|
| 1 | `src/main.ts` | How the app boots, global middleware stack |
| 2 | `src/app.module.ts` | How all modules are wired, middleware config |
| 3 | `src/middlewares/keycloak.middleware.ts` | Auth entry point, JWT validation |
| 4 | `src/guards/roles.guard.ts` | Role enforcement per endpoint |
| 5 | `src/models/vehicles/vehicles.module.ts` | Reference module structure |
| 6 | `src/models/vehicles/vehicles.controller.ts` | Reference controller patterns |
| 7 | `src/models/vehicles/vehicles.service.ts` | Reference service + DB patterns |
| 8 | `submodules/helpers-submodule/provider/tenant-connection.provider.ts` | Multi-tenancy DB switching |
| 9 | `submodules/helpers-submodule/provider/tenant-model.provider.ts` | Per-tenant model factory |
| 10 | `src/utils/socket/socket.gateway.ts` | Real-time WebSocket architecture |

---

## 14. Environment Variables (Key Ones)

| Variable | Purpose |
|---|---|
| `PORT` | App port (default: 5000) |
| `MONGODB_URI` | MongoDB connection string |
| `KEYCLOAK_SERVER_URL` | Keycloak base URL |
| `KEYCLOAK_REALM` | Keycloak realm name |
| `KEYCLOAK_CLIENT_ID` | Client ID registered in Keycloak |
| `KEYCLOAK_CLIENT_SECRET` | Client secret |
| `KEYCLOAK_JWKS_URI` | JWKS endpoint for public key fetching |
| `KEYCLOAK_AUDIENCE` | Expected JWT audience |
| `KEYCLOAK_ISSUER` | Expected JWT issuer |
| `KEYCLOAK_ADMIN_USERNAME` | Keycloak admin username |
| `KEYCLOAK_ADMIN_PASSWORD` | Keycloak admin password |
| `JWT_SECRET` | Secret for signing CSRF tokens |
| `REDIS_HOST` | Redis host |
| `REDIS_PORT` | Redis port |
| `AWS_REGION` | AWS region |
| `AWS_ACCESS_KEY_ID` | AWS access key |
| `AWS_SECRET_ACCESS_KEY` | AWS secret key |

> See `.env.sample` for the full list.

---

*Generated: 2026-03-21*
