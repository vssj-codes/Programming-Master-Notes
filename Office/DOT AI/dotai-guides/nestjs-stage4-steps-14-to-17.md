# Stage 4 — Advanced Patterns (Steps 14–17)

> Project: library-tracker + DotAI backend reference

---

# Step 14 — Multi-tenancy

## The Concept

In a normal app, one codebase → one database. In a multi-tenant app:

```
Client A (TenantID: abc) ──┐
Client B (TenantID: xyz) ──┼──▶  Single NestJS App  ──▶  tenant_abc (MongoDB DB)
Client C (TenantID: def) ──┘                         └──▶  tenant_xyz (MongoDB DB)
                                                      └──▶  tenant_def (MongoDB DB)
```

One app, but each request dynamically connects to a **different MongoDB database** based on who's making the request.

---

## Part 1 — How `tenantId` Gets onto the Request

The Keycloak JWT token contains a `selected_tenant` claim:
```json
{
  "sub": "user-uuid",
  "selected_tenant": "{\"tenant\": \"abc123\"}",
  "resource_access": { ... }
}
```

- **HTTP requests** — middleware extracts it and attaches to `req.tenantId`
- **WebSocket connections** — `SocketStateAdapter` extracts it from the JWT at handshake time and attaches to `socket.tenantId`

---

## Part 2 — The Three-Piece Pattern

```
┌─────────────────────────────────────────────────────┐
│ 1. TenantConnectionProvider  (REQUEST scoped)        │
│    Reads req.tenantId → calls connection.useDb(      │
│    'tenant_abc') → returns tenant-specific Connection│
└───────────────────┬─────────────────────────────────┘
                    │ injects Connection into ▼
┌───────────────────▼─────────────────────────────────┐
│ 2. createTenantModelProvider()                       │
│    Takes tenant Connection + Schema →                │
│    returns tenantConnection.model(name, schema)      │
│    → Mongoose Model bound to THAT tenant's DB        │
└───────────────────┬─────────────────────────────────┘
                    │ injects Model into ▼
┌───────────────────▼─────────────────────────────────┐
│ 3. VehiclesService (or any service)                  │
│    @Inject(PROVIDER.VEHICLE_MODEL)                   │
│    private vehicleModel: Model<VehicleDocument>      │
│    → all queries run against tenant_abc.vehicles     │
└─────────────────────────────────────────────────────┘
```

---

## Part 3 — `TenantConnectionProvider` — Line by Line

```typescript
// submodules/helpers-submodule/provider/tenant-connection.provider.ts
export const TenantConnectionProvider = {
  provide: PROVIDER.TENANT_CONNECTION,
  scope: Scope.REQUEST,          // new instance created per HTTP request
  useFactory: async (request: CustomRequest, connection: Connection) => {
    const dbName = `tenant_${request.tenantId}`;     // e.g. "tenant_abc123"
    const tenantDb = connection.useDb(dbName, { useCache: true });
    return tenantDb;
  },
  inject: [REQUEST, getConnectionToken()],  // NestJS injects the request + base Mongoose connection
};
```

### `Scope.REQUEST` — Why It Matters

| Scope | Created | Destroyed |
|---|---|---|
| `Scope.DEFAULT` (singleton) | Once at app startup | App shuts down |
| `Scope.REQUEST` | Each incoming HTTP request | Request completes |

Without `Scope.REQUEST`, the same DB connection would be shared across all tenants — a critical security flaw.

---

## Part 4 — `createTenantModelProvider` — Line by Line

```typescript
// submodules/helpers-submodule/provider/tenant-model.provider.ts
export function createTenantModelProvider(modelName: string, schema: any, token: string) {
  return {
    provide: token,                            // e.g. PROVIDER.VEHICLE_MODEL
    useFactory: async (tenantConnection: Connection) => {
      return tenantConnection.model(modelName, schema);  // model on THIS tenant's connection
    },
    inject: [PROVIDER.TENANT_CONNECTION],      // depends on TenantConnectionProvider
  };
}
```

Called once per model in the module's `providers` array:
```typescript
// vehicles.module.ts
providers: [
  TenantConnectionProvider,
  createTenantModelProvider(Vehicles.name, VehicleSchema, PROVIDER.VEHICLE_MODEL),
  createTenantModelProvider(Location.name, LocationSchema, PROVIDER.LOCATION_MODEL),
  // ...
]
```

---

## Part 5 — Injecting in the Service

```typescript
@Injectable()
export class VehiclesService {
  constructor(
    @Inject(PROVIDER.VEHICLE_MODEL)
    private vehicleModel: Model<VehicleDocument>,
  ) {}

  async findAll() {
    return this.vehicleModel.find();  // tenant_abc.vehicles for tenant A
                                      // tenant_xyz.vehicles for tenant B
  }
}
```

| | Library Tracker | DotAI |
|---|---|---|
| Model injection | `@InjectModel(Book.name)` | `@Inject(PROVIDER.VEHICLE_MODEL)` |
| Connection | Single shared connection | Per-request tenant connection |

---

## Part 6 — Why Separate DB Per Tenant?

| Approach | How | DotAI |
|---|---|---|
| Shared DB | One DB, every document has `tenantId` field | No |
| Separate DB | `tenant_abc`, `tenant_xyz` each own DB | Yes |

DotAI uses separate DB per tenant:
- Complete data isolation — one bug can't leak another tenant's data
- Independent backup/restore per client
- Can scale or migrate individual tenants independently

---

## Key Takeaways — Step 14

> 1. `selected_tenant` claim in the JWT identifies which DB to use
> 2. `TenantConnectionProvider` — `Scope.REQUEST` — per-request connection to `tenant_{id}`
> 3. `createTenantModelProvider` — creates a Mongoose model on the tenant connection
> 4. Services inject via `@Inject(PROVIDER.X_MODEL)` not `@InjectModel()`
> 5. Separate DB per tenant = complete data isolation between clients

---

---

# Step 15 — WebSockets / Socket.io

## HTTP vs WebSocket

```
HTTP (request-response):
  Client ──── request ────▶ Server
  Client ◀─── response ─── Server
  Connection closes.

WebSocket (persistent, bidirectional):
  Client ──── handshake ───▶ Server
  Client ◀══════════════════▶ Server  ← open connection
  Server pushes data anytime without client asking
```

Use cases in DotAI: live vehicle location updates, real-time alerts, dashboard telemetry.

---

## Part 1 — NestJS Gateway

A **Gateway** is the WebSocket equivalent of a Controller.

```typescript
import {
  WebSocketGateway,
  WebSocketServer,
  SubscribeMessage,
  OnGatewayConnection,
  OnGatewayDisconnect,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';

@WebSocketGateway({
  cors: { origin: '*' },
  namespace: '/socket',   // optional URL namespace
})
export class SocketGateway implements OnGatewayConnection, OnGatewayDisconnect {

  @WebSocketServer()
  server: Server;   // use this to push events to clients

  handleConnection(client: Socket) {
    console.log('Client connected:', client.id);
  }

  handleDisconnect(client: Socket) {
    console.log('Client disconnected:', client.id);
  }

  // Handle incoming event from client
  @SubscribeMessage('joinRoom')
  handleJoinRoom(client: Socket, roomId: string) {
    client.join(roomId);
    return { success: true };
  }

  // Push to all clients in a room (called from a service)
  sendToRoom(roomId: string, event: string, data: any) {
    this.server.to(roomId).emit(event, data);
  }
}
```

---

## Part 2 — Rooms (Tenant Isolation)

DotAI uses Socket.io **rooms** to isolate tenants:

```
Client A (tenant_abc) → joins room "tenant_abc"
Client B (tenant_xyz) → joins room "tenant_xyz"

GPS update arrives for tenant_abc:
  server.to('tenant_abc').emit('locationUpdate', data)
  → only Client A receives it
```

```typescript
handleConnection(client: AuthenticatedSocket) {
  client.join(`tenant_${client.tenantId}`);
}

// from a service:
this.socketGateway.sendToRoom(`tenant_${tenantId}`, 'vehicleLocation', data);
```

---

## Part 3 — Register Gateway in a Module

```typescript
@Module({
  providers: [SocketGateway],
  exports: [SocketGateway],   // export if other modules push events
})
export class SocketModule {}
```

---

## Part 4 — Client Side

```javascript
import { io } from 'socket.io-client';

const socket = io('http://localhost:3000/socket', {
  auth: { token: 'Bearer eyJhbG...' }  // JWT sent at handshake
});

socket.on('vehicleLocation', (data) => updateMap(data));
socket.emit('joinRoom', 'tenant_abc123');
```

---

## Key Takeaways — Step 15

> 1. Gateway = Controller for WebSocket events
> 2. `@WebSocketServer()` — the Socket.io server instance for pushing to clients
> 3. `@SubscribeMessage('event')` — handles incoming event from client
> 4. Rooms — group sockets; push to a room to target a subset (used for tenant isolation)
> 5. `SocketStateAdapter` validates JWT at handshake and attaches `socket.tenantId`

---

---

# Step 16 — Job Queues with Bull

## Why Job Queues?

Some operations are too slow to run inline during an HTTP request:
- Sending emails (200–500ms per SES call)
- Generating large reports
- Processing bulk CSV uploads
- Sending push notifications to thousands of users

```
Without queue: Client waits → slow response
With queue:    Push work onto queue → return 202 immediately → worker processes in background
```

---

## Part 1 — Setup

Bull uses Redis as its backing store.

```bash
npm install @nestjs/bull bull
npm install -D @types/bull
```

```typescript
// app.module.ts
import { BullModule } from '@nestjs/bull';

BullModule.forRoot({
  redis: { host: 'localhost', port: 6379 },
}),
BullModule.registerQueue({ name: 'email' }),
```

---

## Part 2 — Producer (Adding Jobs)

```typescript
import { InjectQueue } from '@nestjs/bull';
import { Queue } from 'bull';

@Injectable()
export class ReportsService {
  constructor(
    @InjectQueue('email') private emailQueue: Queue,
  ) {}

  async requestReport(userId: string, reportType: string) {
    await this.emailQueue.add('send-report', { userId, reportType }, {
      delay: 5000,    // wait 5s before processing
      attempts: 3,    // retry up to 3 times on failure
    });

    return { message: 'Report is being generated' };
  }
}
```

---

## Part 3 — Consumer (Processing Jobs)

```typescript
import { Process, Processor } from '@nestjs/bull';
import { Job } from 'bull';

@Processor('email')                  // listens to 'email' queue
export class EmailProcessor {

  @Process('send-report')            // handles 'send-report' jobs
  async handleSendReport(job: Job<{ userId: string; reportType: string }>) {
    const { userId, reportType } = job.data;
    // slow work here — generate PDF, call SES, update DB
    await this.sesService.sendEmail(userId, reportType);
  }
}
```

Register as a provider:
```typescript
@Module({
  providers: [EmailProcessor],
})
```

---

## Key Takeaways — Step 16

> 1. Queues decouple slow work from the HTTP request/response cycle
> 2. Bull uses Redis — jobs survive app restarts
> 3. `@InjectQueue('name')` + `.add()` — producer adds jobs
> 4. `@Processor('name')` + `@Process('job')` — consumer processes them
> 5. Jobs support `delay`, `attempts`, `backoff` for retry logic

---

---

# Step 17 — MQTT (IoT Messaging)

## What MQTT Is

MQTT is a lightweight publish/subscribe protocol designed for IoT devices (GPS trackers, sensors, vehicles). Devices **publish** readings to topics; the server **subscribes** to those topics.

```
Vehicle GPS ──── publish ────▶ MQTT Broker (AWS IoT Core)
                                      │
                               subscribe ▼
                               NestJS App ──▶ process, store, push via WebSocket
```

HTTP is too heavy for devices publishing every 1–5 seconds. MQTT is purpose-built for high-frequency, low-bandwidth IoT.

---

## Part 1 — Topics

MQTT topics are slash-separated strings:

```
devices/vehicle/abc123/location    ← GPS coordinates
devices/vehicle/abc123/telemetry   ← speed, fuel, temperature
alerts/geofence/abc123             ← geofence events
```

Wildcards in subscriptions:
```
devices/vehicle/+/location   ← '+' matches one segment (any vehicle ID)
devices/#                    ← '#' matches everything below
```

---

## Part 2 — NestJS MQTT Transport

```typescript
// main.ts
app.connectMicroservice({
  transport: Transport.MQTT,
  options: {
    url: 'mqtt://broker.example.com:1883',
    // AWS IoT Core uses mTLS:
    // url: 'mqtts://xxxxx.iot.ap-south-1.amazonaws.com:8883',
    // cert, key, ca for certificate-based auth
  },
});
await app.startAllMicroservices();
```

```typescript
// controller
import { MessagePattern, Payload } from '@nestjs/microservices';

@Controller()
export class IoTController {

  @MessagePattern('devices/vehicle/+/location')
  async handleVehicleLocation(@Payload() data: any) {
    const { vehicleId, lat, lng, tenantId } = data;

    // 1. Save to DB
    await this.locationService.save(vehicleId, lat, lng);

    // 2. Push to connected WebSocket clients
    this.socketGateway.sendToRoom(`tenant_${tenantId}`, 'locationUpdate', data);
  }
}
```

---

## Part 3 — The Full DotAI IoT Flow

```
Vehicle GPS ──MQTT──▶ AWS IoT Core ──▶ NestJS MQTT subscriber
                                              │
                                    ┌─────────▼──────────┐
                                    │  1. Validate data   │
                                    │  2. Save to DB      │
                                    │  3. Check geofence  │
                                    │  4. Bull queue      │
                                    │     (alert email)   │
                                    └─────────┬──────────┘
                                              │
                                    ┌─────────▼──────────┐
                                    │  WebSocket push     │
                                    │  to dashboard       │
                                    └────────────────────┘
```

Steps 15, 16, and 17 all chain together: MQTT receives data → Bull queues the alert → WebSocket pushes the live update.

---

## Key Takeaways — Step 17

> 1. MQTT = publish/subscribe protocol designed for IoT — not HTTP
> 2. Topics are slash-separated strings; `+` and `#` are wildcards
> 3. Devices publish → broker distributes → NestJS subscribes via `@MessagePattern()`
> 4. AWS IoT Core is the broker in DotAI — uses mTLS (certificates) for auth
> 5. MQTT → Bull → WebSocket: the three technologies chain together for real-time IoT

---

## Stage 4 — DotAI vs Library Tracker Summary

| Topic | Library Tracker | DotAI |
|---|---|---|
| Multi-tenancy | Single DB | Per-request DB via `TenantConnectionProvider` |
| WebSockets | Not used | Full Socket.io gateway with tenant room isolation |
| Job Queues | Not used | Bull queues for SES, SNS, report generation |
| MQTT | Not used | AWS IoT Core → `@MessagePattern()` → WebSocket push |

---

*Generated: 2026-03-23 | Project: library-tracker + DotAI*
