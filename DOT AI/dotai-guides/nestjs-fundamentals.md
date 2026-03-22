# NestJS — 5 Core Fundamentals

> A beginner's guide to NestJS using the DotAI Backend codebase as reference.

---

## Table of Contents

1. [What is NestJS?](#1-what-is-nestjs)
2. [Modules](#2-modules)
3. [Controllers](#3-controllers)
4. [Services](#4-services)
5. [Dependency Injection](#5-dependency-injection)

---

## 1. What is NestJS?

### The Problem Before NestJS

Express.js is very minimal — it gives you routing and middleware, that's it. With complete freedom comes inconsistency at scale:
- One dev puts logic in the route file
- Another creates service files
- No consistent pattern for organizing modules
- No built-in dependency injection
- Hard to test, hard to onboard new devs

### What NestJS Does

NestJS is a framework **built on top of Express** that brings **structure and conventions** to Node.js backends. It takes inspiration from **Angular** (frontend) and **Spring Boot** (Java backend).

| Problem | NestJS Solution |
|---|---|
| How do I organize code? | **Modules** |
| How do I handle HTTP routes? | **Controllers** |
| Where does business logic go? | **Services** |
| How do classes share dependencies? | **Dependency Injection** |
| How do I protect routes? | **Guards** |
| How do I validate input? | **Pipes** |
| How do I handle errors? | **Exception Filters** |

### The Big Idea — Decorators

NestJS uses **TypeScript decorators** heavily. A decorator is a function that adds metadata or behavior to a class or method.

```typescript
@Controller('vehicles')        // this class handles /vehicles routes
export class VehiclesController {

  @Get()                       // this method handles GET /vehicles
  findAll() { ... }

  @Post()                      // this method handles POST /vehicles
  create() { ... }
}
```

No manual route registration. NestJS reads the decorators and wires everything up automatically.

### How It Sits on Express

```
Your NestJS Code
      ↓
NestJS Framework (structure, decorators, DI)
      ↓
Express.js (actual HTTP server)
      ↓
Node.js
```

### In This Codebase

- **60+ modules** — one per feature (vehicles, assets, geofence, etc.)
- Each module has its own controller, service, and schema
- Everything wired together in `src/app.module.ts` — the root module

---

## 2. Modules

### The Core Idea

A **Module** is a way to group related code together. Think of it as a **feature folder with a manifest file**.

Every NestJS app has at least one module — the **Root Module**. From there, it's composed of many feature modules.

```
App
├── Root Module (app.module.ts)
│   ├── Vehicles Module
│   ├── Assets Module
│   ├── Geofence Module
│   ├── Users Module
│   └── ... 60+ more
```

### What a Module Looks Like

```typescript
@Module({
  imports: [],      // other modules this module depends on
  controllers: [],  // controllers that belong to this module
  providers: [],    // services, guards, etc. that belong to this module
  exports: [],      // what this module exposes to other modules
})
export class VehiclesModule {}
```

### The Four Properties Explained

#### `controllers`
The HTTP layer. Receives requests, returns responses.
```typescript
controllers: [VehiclesController]
```

#### `providers`
Everything that can be injected — services, guards, database models, etc.
```typescript
providers: [
  VehiclesService,
  AuthService,
  CommonService,
  LocationService,
  TenantConnectionProvider,
  createTenantModelProvider(Vehicles.name, VehicleSchema, PROVIDER.VEHICLE_MODEL),
  createTenantModelProvider(Location.name, LocationSchema, PROVIDER.LOCATION_MODEL),
  // ...
]
```

#### `imports`
Other modules whose exported providers you need to use.
```typescript
imports: [
  MongooseModule.forFeature([...]),  // registers MongoDB models
  SESHandlerModule,                  // email sending
  GeofenceModule,                    // geofence logic
]
```

#### `exports`
What this module makes available to other modules that import it.
```typescript
exports: [VehiclesService]
// If not exported, it's private to this module
```

### Real File — `src/models/vehicles/vehicles.module.ts`

```typescript
@Module({
  imports: [
    MongooseModule.forFeature([{ name: ComputeNode.name, schema: ComputeNodeSchema }]),
    SESHandlerModule,
    GeofenceModule,
  ],
  controllers: [VehiclesController],
  providers: [
    VehiclesService,
    AuthService,
    CommonService,
    LocationService,
    TenantConnectionProvider,
    createTenantModelProvider(Vehicles.name, VehicleSchema, PROVIDER.VEHICLE_MODEL),
    createTenantModelProvider(Location.name, LocationSchema, PROVIDER.LOCATION_MODEL),
    // ...more models
  ],
  exports: [VehiclesService],
})
export class VehiclesModule {}
```

### The Root Module — `src/app.module.ts`

The **entry point** of the entire app. Imports every feature module:

```typescript
imports: [
  VehiclesModule,
  TrailersModule,
  GeofenceModule,
  UsersModule,
  // ... 55+ more
]
```

Think of `app.module.ts` as the **table of contents** of the entire application.

### Mental Model

```
app.module.ts  (root — imports everything)
│
├── vehicles.module.ts
│     ├── VehiclesController   (handles HTTP)
│     ├── VehiclesService      (business logic)
│     └── Mongoose models      (DB access)
│
├── geofence.module.ts
│     ├── GeofenceController
│     └── GeofenceService
│
└── ... 60+ more modules
```

> **Key Takeaway:** A Module = a feature. Controller + Service + DB models, all grouped together and registered in one place.

---

## 3. Controllers

### What is a Controller?

A **Controller** is the HTTP layer of your app. Its only job is:
- Receive an HTTP request
- Call the right service method
- Return a response

It has **zero business logic**. No DB queries, no calculations. Just routing.

### Basic Structure

```typescript
@Controller('vehicles')
export class VehiclesController {

  constructor(private vehiclesService: VehiclesService) {}

  @Get()               // GET /vehicles
  findAll() {
    return this.vehiclesService.getAllVehicles();
  }

  @Get('/:id')         // GET /vehicles/123
  findOne(@Param('id') id: string) {
    return this.vehiclesService.getVehicleById(id);
  }

  @Post()              // POST /vehicles
  create(@Body() dto: CreateVehicleDto) {
    return this.vehiclesService.create(dto);
  }
}
```

### Class-Level Decorators

```typescript
@ApiTags('Vehicle')              // groups under "Vehicle" in Swagger docs
@ApiBearerAuth()                 // tells Swagger this needs a Bearer token
@Controller(ERouteName.VEHICLES) // base route = /vehicles
export class VehiclesController {
  constructor(private vehicleService: VehiclesService) {}
```

### Method-Level Decorators

| Decorator | What it does |
|---|---|
| `@Get()`, `@Post()`, `@Patch()`, `@Delete()` | HTTP method |
| `@Param('id')` | Extracts `:id` from the URL path |
| `@Query()` | Extracts `?key=value` from the URL |
| `@Body()` | Extracts the request body (JSON) |
| `@Req()` | Gives you the full raw request object |
| `@Res()` | Gives you the raw response object |
| `@User()` | Custom decorator — extracts user info from JWT |
| `@UseGuards(...)` | Applies guards to this method |
| `@Roles(...)` | Declares which roles can access this method |
| `@Version(...)` | Applies API versioning to this method |

### Route Table — Vehicles Controller

| Method | URL | Who can access | CSRF required? |
|---|---|---|---|
| `GET` | `/vehicles` | all roles | No |
| `GET` | `/v1/vehicles` | all except operator | No |
| `GET` | `/vehicles/export` | all except operator | No |
| `GET` | `/vehicles/location/:id` | all except operator | No |
| `GET` | `/vehicles/:id` | all roles | No |
| `POST` | `/vehicles` | admin, installer | Yes |
| `PATCH` | `/vehicles/:id` | admin, installer | Yes |
| `DELETE` | `/vehicles/multi` | admin, installer | Yes |
| `DELETE` | `/vehicles/:id` | admin, installer | Yes |

**Pattern: reads are open to all roles, writes are restricted to admin/installer.**

### Two Versions of the Same Endpoint

```typescript
@Get()                    // GET /vehicles (default, simpler)
getAllVehicles(...)

@Version(CURRENT_VERSION) // GET /v1/vehicles (paginated)
@Get()
getAllVehiclesV1(...)
```

### The `@User()` Custom Decorator

```typescript
async create(
  @Body() createVehiclePayload: CreateVehicleDto,
  @User() user: userDataDTO,   // extracts logged-in user from JWT
)
```

Defined in `src/decorators/user.decorator.ts`. Used in every mutating endpoint to record `createdBy` / `updatedBy`.

### The `@OnEvent()` Handler

```typescript
@OnEvent(ENestEventTopics.EXPORT_VEHICLE_DATA)
async exportVehiclesData(payload: { ids: string; user: userDataDTO }) {
  await this.vehicleService.exportVehiclesData(ids, user);
}
```

**Not an HTTP endpoint.** It's an event listener — fires asynchronously when another part of the app emits `EXPORT_VEHICLE_DATA`. Used for heavy operations so the HTTP request doesn't have to wait.

### Standard Method Structure

Every method follows this exact pattern:

```typescript
@Get('/:id')
@UseGuards(RolesGuard)
@Roles('admin', 'viewer', ...)
async getVehiclesById(@Res() res, @Param('id') vehicleId) {
  try {
    const { success, data, message } = await this.vehicleService.getVehicleById(vehicleId);

    if (!success) {
      return res.status(400).send(formatErrorResponse(...));  // business failure
    }
    return res.status(200).send(formatResponse(data, message)); // success

  } catch (err) {
    return res.status(500).send(formatErrorResponse(err, 'Something went wrong!')); // crash
  }
}
```

Three outcomes every time: **success (200)**, **business failure (400)**, **unexpected crash (500)**.

> **Key Takeaway:** Controllers only route — all logic is in the service. Guards sit on top of every mutating endpoint. `@OnEvent()` makes controllers act as event listeners too, not just HTTP handlers.

---

## 4. Services

### What is a Service?

A **Service** is where all business logic lives. No HTTP knowledge, no request/response — just pure logic and DB operations.

> The controller says *what to do*. The service *does it*.

### Constructor — Dependency Injection

```typescript
@Injectable()
export class VehiclesService {
  constructor(
    @Inject(PROVIDER.VEHICLE_MODEL)
    private vehicleModel: Model<VehiclesDocuments>,   // Mongoose model (tenant DB)

    @Inject(PROVIDER.LOCATION_MODEL)
    private locationModel: Model<LocationDocument>,   // another Mongoose model

    private commonService: CommonService,             // shared utility service
    private locationService: LocationService,         // location feature service
    private sesHandlerService: SESHandlerService,     // AWS SES email service
  ) {}
```

### DB Access Patterns

#### 1. Simple Find (read-only, fast)
```typescript
this.vehicleModel.find({ deletedAt: null }).sort({ createdAt: -1 }).lean()
// .lean() returns plain JS objects instead of Mongoose documents — faster for reads
```

#### 2. Aggregation (joins, sorting, grouping)
```typescript
this.vehicleModel.aggregate([
  { $match: { _id: new Types.ObjectId(id) } },      // filter

  { $lookup: {                                       // JOIN locations
      from: 'locations',
      localField: '_id',
      foreignField: 'linkedTo',
      as: 'locationData'
  }},

  { $lookup: {                                       // JOIN transit-tag
      from: 'transit-tag',
      localField: 'locationData.entityId',
      foreignField: '_id',
      as: 'transitTagData'
  }},

  { $unwind: { path: '$transitTagData',             // flatten array → object
               preserveNullAndEmptyArrays: true } }
])
// $lookup = LEFT JOIN,  $unwind = flatten one-element array into object
```

#### 3. Create New Document
```typescript
const vehiclePayload = { ...payload, createdBy: user, updatedBy: user };
const createdVehicle = new this.vehicleModel(vehiclePayload);
await createdVehicle.save();
```

#### 4. Update and Return Updated Document
```typescript
this.vehicleModel.findOneAndUpdate(
  { _id: new Types.ObjectId(id) },
  { $set: { ...payload, updatedAt: Date.now(), updatedBy: user } },
  { new: true }   // ← return the updated document, not the old one
).lean()
```

#### 5. Soft Delete then Hard Delete
```typescript
// Step 1: Soft delete — record who deleted it
await this.vehicleModel.findOneAndUpdate(
  { _id: id },
  { $set: { deletedAt: Date.now(), deletedBy: user } },
  { new: true }
)

// Step 2: Hard delete — remove from DB
await this.vehicleModel.findOneAndDelete({ _id: id })

// Step 3: Archive linked records
await this.locationService.archive(id, 'linkedTo', user)
```

#### 6. Bulk Operations
```typescript
// Update many at once
await this.vehicleModel.updateMany(
  { _id: { $in: vehicleIds } },
  { $set: { deletedAt: new Date(), deletedBy: user } }
)

// Delete many at once
await this.vehicleModel.deleteMany({ _id: { $in: vehicleIds } })
```

### Full Pagination Pattern (v1 endpoints)

```typescript
// 1. Build where clause
const whereClause = { deletedAt: null };
if (searchText) whereClause.$or = [{ name: regex }, { transitId: regex }, ...]

// 2. Count total for pagination math
const totalDocuments = await this.vehicleModel.find(whereClause).count();

// 3. Aggregation with sort + skip + limit
const data = await this.vehicleModel.aggregate([
  { $match: whereClause },
  { $sort: { [sortFrom]: sort } },
  { $skip: offset },
  { $limit: pageSize }
])

// 4. Return data + metaData
return {
  success: true,
  data,
  metaData: {
    totalCount,
    totalPages,
    hasPreviousPage,
    hasNextPage,
    links: { self, first, prev, next, last }
  }
}
```

### CSV Export via Email

```typescript
// 1. Query and shape data with $project (rename fields for CSV headers)
const vehicleData = await this.vehicleModel.aggregate([
  { $match: whereClause },
  { $project: {
      'Transit ID': '$transitId',
      'Name': '$name',
      'Status': '$status',
  }}
])

// 2. Convert to CSV
const body = json2csv(vehicleData)

// 3. Send via AWS SES
await this.sesHandlerService.sendRawEmail(fromEmail, toEmail, body, true, 'Vehicle')
```

### DB Patterns — Summary Table

| Pattern | Method | When used |
|---|---|---|
| `.find().lean()` | `getAllVehicles` | Simple reads, no modification |
| `.aggregate([...])` | `getVehicleById` | Joins, sorting, grouping, projections |
| `.findOneAndUpdate({new:true})` | `update` | Update and return updated doc |
| `new Model(data).save()` | `create` | Insert new document |
| `updateMany` / `deleteMany` | `handleBulkDelete` | Bulk operations |
| Two-step soft + hard delete | `deleteVehicleById` | All deletes in this codebase |

> **Key Takeaway:** The service is the brain. Controller handles HTTP, service handles everything else. Every method returns `{ success, data, message }` so the controller always knows what happened.

---

## 5. Dependency Injection (DI)

### The Problem It Solves

Without DI, you'd manually create and pass every dependency:

```typescript
// Without DI — you manage everything manually
const locationService = new LocationService(locationModel);
const sesService = new SESHandlerService(awsConfig);
const vehicleService = new VehiclesService(vehicleModel, locationService, sesService);
const controller = new VehiclesController(vehicleService);
```

This gets out of hand fast. **DI flips this around** — you declare what you need, NestJS figures out how to build it.

### How It Works — Three Steps

#### Step 1 — Mark a class as injectable
```typescript
@Injectable()
export class VehiclesService { ... }
```
Tells NestJS: *"I can be created and injected by the framework."*

#### Step 2 — Declare what you need in the constructor
```typescript
export class VehiclesService {
  constructor(
    private locationService: LocationService,
    private sesHandlerService: SESHandlerService,
  ) {}
}
```
NestJS reads the TypeScript types and knows what to inject.

#### Step 3 — Register in the module
```typescript
@Module({
  providers: [VehiclesService, LocationService, SESHandlerService]
})
```
Once registered, NestJS builds it and injects it wherever needed.

### The DI Container — Mental Model

Think of NestJS as a **registry of objects**:

```
NestJS Container
  ├── VehiclesService    (singleton — built once, shared within module)
  ├── LocationService    (singleton — built once, shared)
  ├── SESHandlerService  (singleton — built once, shared)
  └── VehicleModel       (REQUEST scoped — new instance per request)
```

When a controller needs a service, NestJS:
1. Looks up the service in the container
2. Sees what it depends on
3. Builds those dependencies first (recursively)
4. Then builds the service with everything wired in
5. Injects the result into the controller

**You never call `new` — NestJS does it all.**

### Scopes — How Long Does an Instance Live?

| Scope | Lifetime | Used for |
|---|---|---|
| `DEFAULT` (Singleton) | Entire app lifetime | Most services |
| `REQUEST` | New instance per HTTP request | `TenantConnectionProvider` |
| `TRANSIENT` | New instance every time it's injected | Rare |

The multi-tenancy system depends entirely on `REQUEST` scope:

```typescript
// tenant-connection.provider.ts
{
  provide: PROVIDER.TENANT_CONNECTION,
  scope: Scope.REQUEST,        // new instance per request
  useFactory: async (req) => {
    const tenantId = req.tenantId;
    return connection.useDb(`tenant_${tenantId}`);
  }
}
```

Every request gets its own DB connection → tenant isolation.

### Custom Injection with `@Inject()` Token

For Mongoose models, TypeScript types aren't enough (multiple models all have type `Model<something>`). String tokens solve this:

```typescript
// 1. Define tokens in an enum
export enum PROVIDER {
  VEHICLE_MODEL = 'VEHICLE_MODEL',
  LOCATION_MODEL = 'LOCATION_MODEL',
}

// 2. Register in module with that token
createTenantModelProvider(Vehicles.name, VehicleSchema, PROVIDER.VEHICLE_MODEL)
// Which creates:
{
  provide: PROVIDER.VEHICLE_MODEL,
  useFactory: (tenantConn) => tenantConn.model('Vehicle', VehicleSchema),
  inject: [PROVIDER.TENANT_CONNECTION]
}

// 3. Inject in service using the token
constructor(
  @Inject(PROVIDER.VEHICLE_MODEL)
  private vehicleModel: Model<VehiclesDocuments>,
)
```

### Full Chain — Tracing a Request

```
HTTP Request arrives (tenant: "abc123")
  │
  ├─ TenantConnectionProvider (REQUEST scoped)
  │    reads req.tenantId = "abc123"
  │    calls connection.useDb("tenant_abc123")
  │    provides: PROVIDER.TENANT_CONNECTION
  │
  ├─ createTenantModelProvider (VEHICLE_MODEL)
  │    injects PROVIDER.TENANT_CONNECTION
  │    calls tenantConn.model("Vehicle", VehicleSchema)
  │    provides: PROVIDER.VEHICLE_MODEL
  │
  └─ VehiclesService
       @Inject(PROVIDER.VEHICLE_MODEL) → tenant-specific vehicle model ✓
       private locationService → LocationService singleton ✓
       private sesHandlerService → SESHandlerService singleton ✓
```

All of this happens automatically before your controller method even runs.

> **Key Takeaway:** `@Injectable()` marks something as available. `providers[]` in the module registers it. NestJS builds and connects everything automatically. `@Inject()` with a token is used when TypeScript types aren't enough (like Mongoose models).

---

## Quick Reference

### The 5 Concepts at a Glance

| Concept | File Location | One Line Summary |
|---|---|---|
| **NestJS** | `src/main.ts` | Structured framework on top of Express |
| **Modules** | `src/models/vehicles/vehicles.module.ts` | Feature grouping — controller + service + DB models |
| **Controllers** | `src/models/vehicles/vehicles.controller.ts` | HTTP routing only — no logic |
| **Services** | `src/models/vehicles/vehicles.service.ts` | All business logic and DB queries |
| **DI** | `submodules/helpers-submodule/provider/` | Framework wires everything together automatically |

### Standard Patterns Across the Codebase

```
Every module   → controller + service + module file + dto/ + interface/
Every response → { success: boolean, data: any, message: string }
Every read     → filter { deletedAt: null }
Every write    → record createdBy / updatedBy from @User() decorator
Every delete   → soft delete first (deletedAt) then hard delete
Every v1 GET   → full pagination with metaData
```

---

*Generated: 2026-03-22 | Reference codebase: DotAI Backend*
