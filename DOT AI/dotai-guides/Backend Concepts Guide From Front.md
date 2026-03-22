# Backend Concepts Guide: From Frontend to NestJS Backend

> A beginner-friendly, technically accurate guide for a frontend developer transitioning to backend engineering, using real examples from your codebase.

---

## Table of Contents

1. [Multi-Tenant SaaS NestJS Backend](#1-multi-tenant-saas-nestjs-backend)
2. [Providers in NestJS](#2-providers-in-nestjs)
3. [Controllers in NestJS](#3-controllers-in-nestjs)
4. [@Inject Decorator](#4-inject-decorator)
5. [Controller-Service Pattern](#5-controller-service-pattern)
6. [Bearer Token Authentication](#6-bearer-token-authentication)
7. [Refresh Token](#7-refresh-token)
8. [Gateway in NestJS](#8-gateway-in-nestjs)
9. [Socket.IO](#9-socketio)
10. [Pub-Sub Model](#10-pub-sub-model)
11. [Topics in Pub-Sub](#11-topics-in-pub-sub)
12. [AWS MQTT](#12-aws-mqtt)
13. [MongoDB Aggregation Pipeline](#13-mongodb-aggregation-pipeline)
14. [How Everything Connects: Architecture Diagram](#14-how-everything-connects)
15. [Full Request Flow Example](#15-full-request-flow-example)
16. [Quick Cheat Sheet](#16-quick-cheat-sheet)

---

## 1. Multi-Tenant SaaS NestJS Backend

### Simple Definition
Imagine an apartment building. One building (your backend), many tenants (companies). Each company gets its own "apartment" (database) inside the same building. They share the elevator and lobby (API code), but can never enter each other's rooms (data isolation).

### Why It Exists
Without multi-tenancy, you'd need to deploy a separate backend for every customer. That means 100 customers = 100 servers, 100 databases, 100 deployment pipelines. Multi-tenancy lets you run one codebase that serves all customers while keeping their data completely separate.

### How It Works Conceptually
```
Customer A logs in  --->  JWT has "tenantId: company_a"
Customer B logs in  --->  JWT has "tenantId: company_b"

Same API server, but:
  Customer A's queries hit database: tenant_company_a
  Customer B's queries hit database: tenant_company_b
```

### How It Appears in Your NestJS Project

**Step 1 - Middleware extracts tenant** (`submodules/helpers-submodule/middlewares/tenant-middelware.ts`):
```typescript
// Simplified from your actual middleware
@Injectable()
export class TenantsMiddleware implements NestMiddleware {
  async use(req: Request, res: Response, next: NextFunction) {
    // 1. Decode the JWT token from the Authorization header
    const decodedToken = parseJwt(req.headers.authorization);

    // 2. Extract the tenant name from the token's claims
    const parsedTenant = JSON.parse(decodedToken.selected_tenant);
    const tenantId = parsedTenant?.tenant;  // e.g. "company_abc"

    // 3. Verify tenant exists in the master database
    const isTenantExist = await this.tenantModel.findOne({ tenantName: tenantId });
    if (!isTenantExist) {
      return res.status(404).send({ success: false, message: 'Tenant Not found!' });
    }

    // 4. Attach tenant info to request object for downstream use
    req.tenantId = isTenantExist.tenantName;
    req.user = { userName: decodedToken.preferred_username, email: decodedToken.email };
    next();  // pass to next middleware
  }
}
```

**Step 2 - Provider switches database** (`submodules/helpers-submodule/provider/tenant-connection.provider.ts`):
```typescript
export const TenantConnectionProvider = {
  provide: PROVIDER.TENANT_CONNECTION,
  scope: Scope.REQUEST,  // New instance per request!
  useFactory: async (request, connection) => {
    const dbName = `tenant_${request.tenantId}`;  // "tenant_company_abc"
    return connection.useDb(dbName, { useCache: true });  // Switch to tenant's DB
  },
  inject: [REQUEST, getConnectionToken()],
};
```

**Step 3 - Models are scoped to that tenant's DB** (`submodules/helpers-submodule/provider/tenant-model.provider.ts`):
```typescript
export function createTenantModelProvider(modelName, schema, token) {
  return {
    provide: token,               // e.g. PROVIDER.ITEM_MODEL
    useFactory: (tenantConnection) => {
      return tenantConnection.model(modelName, schema);  // Model talks to tenant DB only
    },
    inject: [PROVIDER.TENANT_CONNECTION],  // Uses the tenant-specific connection
  };
}
```

### Frontend Analogy
Think of it like React Context. The `TenantsMiddleware` is like a `<TenantProvider>` that wraps the entire request. Every component (service) downstream automatically receives the correct tenant context without needing to pass it explicitly.

### Common Mistakes
- **Forgetting `Scope.REQUEST`**: Without request scope, the provider would be a singleton, meaning all tenants share the same DB connection (data leak!)
- **Querying without tenant context**: If you call a service method outside the HTTP request lifecycle (e.g., from a cron job), there's no `req.tenantId` and it will crash
- **Not adding `TenantConnectionProvider` to module providers**: Every module that uses tenant-scoped models must include it

---

## 2. Providers in NestJS

### Simple Definition
A provider is anything that can be **injected** (given) to other classes. Think of it like props in React, but automatic. Instead of manually passing dependencies, NestJS figures out what each class needs and gives it to them.

### Why It Exists
Without providers/DI (Dependency Injection), every service would manually create its own dependencies:
```typescript
// BAD: Without DI
class ItemsService {
  private db = new MongoConnection('mongodb://...');  // hardcoded
  private emailer = new SESService('key-123');         // hardcoded
}
```
This makes testing impossible and couples everything together. Providers solve this by letting NestJS manage creation and wiring.

### How It Works Conceptually
```
You tell NestJS:  "Here's how to create X"    (Provider)
NestJS says:      "OK, who needs X?"            (looks at constructors)
NestJS does:      "Here you go" (auto-injects)  (Dependency Injection)
```

### Types of Providers in Your Project

**Type 1 - Class Provider (most common):**
```typescript
// Just listing a service class - NestJS auto-creates it
@Module({
  providers: [ItemsService],  // NestJS creates new ItemsService() and injects its deps
})
```

**Type 2 - Factory Provider (custom creation logic):**
```typescript
// From your redis.providers.ts
export const redisProviders: Provider[] = [
  {
    useFactory: () => new Redis(redisConfig()),  // Custom creation function
    provide: REDIS_SUBSCRIBER_CLIENT,            // Token to identify this provider
  },
  {
    useFactory: () => new Redis(redisConfig()),
    provide: REDIS_PUBLISHER_CLIENT,
  },
];
```

**Type 3 - Tenant Model Provider (dynamic factory):**
```typescript
// From your tenant-model.provider.ts
createTenantModelProvider(Item.name, ItemSchema, PROVIDER.ITEM_MODEL)
// This returns a factory provider that creates a Mongoose model
// connected to the current tenant's database
```

### How Providers Appear in a Module
From your `src/models/items/items.module.ts`:
```typescript
@Module({
  providers: [
    ItemsService,                    // Class provider
    AuthService,                     // Class provider
    TenantConnectionProvider,        // Factory provider (tenant DB connection)
    createTenantModelProvider(       // Factory provider (tenant-scoped model)
      Item.name, ItemSchema, PROVIDER.ITEM_MODEL
    ),
    createTenantModelProvider(
      AssetTags.name, AssetTagSchema, PROVIDER.ASSET_TAG_MODEL
    ),
  ],
})
```

### Frontend Analogy
Providers are like React's Context API + dependency injection frameworks. `useFactory` is like a `useMemo` that creates a value once. The `provide` token is like the context itself, and `inject` is like `useContext`.

### Common Mistakes
- **Forgetting to add provider to module**: If `ItemsService` isn't in the `providers` array, NestJS throws `Nest can't resolve dependencies`
- **Circular dependencies**: Service A injects Service B, but Service B also injects Service A. Use `forwardRef()` to fix
- **Not understanding scope**: Default is singleton (one instance). `Scope.REQUEST` creates a new instance per request (needed for multi-tenancy)

---

## 3. Controllers in NestJS

### Simple Definition
A controller is the "receptionist" of your backend. When an HTTP request arrives, the controller decides which method handles it, extracts the data from the request, calls the right service, and sends back the response. It does NOT contain business logic.

### Why It Exists
Without controllers, you'd have one massive file handling all routes. Controllers organize your API by feature/resource, keeping route handling separate from business logic.

### How It Works Conceptually
```
Client Request: POST /items { name: "Widget" }
                    |
                    v
Controller:  @Post() create(@Body() payload)  --> receives request
                    |
                    v
             calls this.itemsService.create(payload)  --> delegates to service
                    |
                    v
             returns res.status(200).send(result)  --> sends response
```

### How It Appears in Your Project
From `src/models/items/items.controller.ts`:
```typescript
@ApiTags('Items')            // Swagger grouping
@ApiBearerAuth()             // Requires JWT token
@Controller('/items')        // Base route: /items
export class ItemsController {
  constructor(private itemsService: ItemsService) {}  // Service injected automatically

  // GET /items - fetch all items
  @Get()
  @UseGuards(RolesGuard)
  @Roles('admin', 'manager', 'installer', 'viewer', 'groupadmin', 'developer')
  public async getAllItems(@Res() res: Response) {
    try {
      const { success, data, error, message } = await this.itemsService.getAllItems();
      if (!success) {
        return res.status(HttpStatus.BAD_REQUEST).send(formatErrorResponse(error, message));
      }
      return res.status(HttpStatus.OK).send(formatResponse(data, message));
    } catch (err) {
      return res.status(HttpStatus.INTERNAL_SERVER_ERROR)
        .send(formatErrorResponse(err, 'Failed to fetch Items'));
    }
  }

  // GET /items/:id - fetch one item by ID
  @Get('/:id')
  @UseGuards(RolesGuard)
  @Roles('admin', 'manager', 'installer', 'viewer', 'groupadmin', 'developer')
  public async getItemById(@Res() res: Response, @Param('id') ItemId: ObjectId) {
    try {
      const { success, data, error, message } = await this.itemsService.findItemById(ItemId);
      if (!success) {
        return res.status(HttpStatus.BAD_REQUEST).send(formatErrorResponse(error, message));
      }
      return res.status(HttpStatus.OK).send(formatResponse(data, message));
    } catch (err) {
      return res.status(HttpStatus.INTERNAL_SERVER_ERROR)
        .send(formatErrorResponse(err, 'Failed to fetch Items'));
    }
  }

  // POST /items - create new item (requires CSRF + admin/installer role)
  @Post()
  @UseGuards(CSRFGuard)
  @UseGuards(RolesGuard)
  @Roles('admin', 'installer')
  public async create(
    @Res() res: Response,
    @Body() createPayload: CreateItemDto,   // Request body auto-validated
    @User() user: userDataDTO,              // Decoded JWT user info
  ) {
    // ... calls service and returns response
  }
}
```

### Key Decorators Explained
| Decorator | Purpose | Frontend Equivalent |
|---|---|---|
| `@Controller('/items')` | Sets the base URL path | Like `<Route path="/items">` |
| `@Get()`, `@Post()`, `@Put()`, `@Delete()` | HTTP method handlers | Like `fetch()` method param |
| `@Param('id')` | Extracts URL parameter | Like `useParams().id` in React Router |
| `@Body()` | Extracts request body | Like reading `JSON.parse(request.body)` |
| `@Query()` | Extracts query string | Like `useSearchParams()` |
| `@User()` | Custom decorator - extracts decoded JWT | Like reading from auth context |
| `@UseGuards(RolesGuard)` | Runs authorization check before method | Like a `<ProtectedRoute>` wrapper |
| `@Roles('admin')` | Metadata for the guard to check | Like `requiredRole` prop |

### Common Mistakes
- **Putting business logic in controllers**: Controllers should ONLY handle request/response. All logic belongs in services
- **Not handling errors**: Always wrap in try/catch. Unhandled errors crash the request
- **Forgetting guards**: Without `@UseGuards(RolesGuard)`, the `@Roles()` decorator does nothing on its own

---

## 4. @Inject Decorator

### Simple Definition
`@Inject()` tells NestJS: "I need this specific thing, and here's the name tag to find it." It's used when NestJS can't automatically figure out what to inject (because it's not a simple class).

### Why It Exists
NestJS can auto-inject classes (it reads the TypeScript type). But for custom providers identified by **string tokens** (like `PROVIDER.ITEM_MODEL`), NestJS doesn't know what type to look for. `@Inject()` says: "find the provider registered under THIS token."

### How It Works Conceptually
```
// Auto-injection (no @Inject needed) - NestJS reads the type
constructor(private itemsService: ItemsService) {}
// NestJS thinks: "I need an ItemsService. Let me find that class provider."

// Manual injection (@Inject needed) - token-based lookup
constructor(@Inject(PROVIDER.ITEM_MODEL) private itemModel: Model<Item>) {}
// NestJS thinks: "I need the provider registered as 'PROVIDER.ITEM_MODEL' string token."
```

### How It Appears in Your Project
From `src/models/items/items.service.ts`:
```typescript
@Injectable()
export class ItemsService {
  constructor(
    // TOKEN-BASED injection (needs @Inject because they're custom providers)
    @Inject(PROVIDER.ITEM_MODEL) private itemModel: Model<ItemDocument>,
    @Inject(PROVIDER.STRUCTURE_MODEL) private structureModel: Model<StructureDocument>,
    @Inject(PROVIDER.LOCATION_MODEL) private locationModel: Model<LocationDocument>,
    @Inject(PROVIDER.ASSET_TAG_MODEL) private assetTagModel: Model<AssetTagsDocument>,
    @Inject(PROVIDER.ASSET_TAG_ITEM_MODEL) private assetTagsItemModel: Model<AssetTagsItemsDocument>,

    // CLASS-BASED injection (no @Inject needed - NestJS reads the type)
    private commonService: CommonService,
    private eventEmitter: EventEmitter2,
    private sESHandlerService: SESHandlerService,
    private awsMqttService: AWSMQTTService,
  ) {}
}
```

### Where Does `PROVIDER.ITEM_MODEL` Come From?
From `submodules/enums-interfaces-submodule/enums/providerEnum.ts`:
```typescript
export enum PROVIDER {
  TENANT_CONNECTION = 'TENANT_CONNECTION',
  ITEM_MODEL = 'ITEM_MODEL',
  ASSET_TAG_MODEL = 'ASSET_TAG_MODEL',
  LOCATION_MODEL = 'LOCATION_MODEL',
  // ... 130+ more tokens
}
```

### The Connection: Token Registration -> Token Injection
```
Module registers:  createTenantModelProvider(Item.name, ItemSchema, PROVIDER.ITEM_MODEL)
                                                                    ^^^^^^^^^^^^^^^^^^
                                                                    This token

Service injects:   @Inject(PROVIDER.ITEM_MODEL) private itemModel
                           ^^^^^^^^^^^^^^^^^^
                           Same token - NestJS matches them
```

### Frontend Analogy
```tsx
// React Context equivalent
const ItemModelContext = createContext(null);  // = PROVIDER.ITEM_MODEL token

// Provider (in module)
<ItemModelContext.Provider value={mongooseModel}>  // = createTenantModelProvider(...)

// Consumer (in service)
const itemModel = useContext(ItemModelContext);     // = @Inject(PROVIDER.ITEM_MODEL)
```

### Common Mistakes
- **Using @Inject for class-based injection**: `@Inject(ItemsService)` is redundant. Just use `private itemsService: ItemsService`
- **Mismatched tokens**: If the module registers `PROVIDER.ITEM_MODEL` but the service injects `PROVIDER.ITEMS_MODEL` (plural), NestJS throws a dependency error
- **Forgetting `@Injectable()`**: The class receiving injections must be decorated with `@Injectable()`

---

## 5. Controller-Service Pattern

### Simple Definition
Split responsibilities: the **Controller** handles HTTP (what comes in, what goes out), the **Service** handles business logic (what happens in between). Think of it as: the controller is the waiter, the service is the chef.

### Why It Exists
Mixing HTTP handling with business logic creates untestable, unmaintainable code. By separating them:
- Services can be tested without HTTP
- Services can be reused (called from MQTT handler, WebSocket, cron job, etc.)
- Controllers stay thin and predictable

### The Complete Chain in Your Project

**1. Controller receives request** (`items.controller.ts`):
```typescript
@Get('/:id')
@UseGuards(RolesGuard)
@Roles('admin', 'manager', 'installer', 'viewer')
public async getItemById(@Res() res: Response, @Param('id') ItemId: ObjectId) {
  try {
    const { success, data, error, message } = await this.itemsService.findItemById(ItemId);
    if (!success) {
      return res.status(HttpStatus.BAD_REQUEST).send(formatErrorResponse(error, message));
    }
    return res.status(HttpStatus.OK).send(formatResponse(data, message));
  } catch (err) {
    return res.status(HttpStatus.INTERNAL_SERVER_ERROR)
      .send(formatErrorResponse(err, 'Failed to fetch Items'));
  }
}
```

**2. Service handles business logic** (`items.service.ts`):
```typescript
public async findItemById(id: ObjectId): Promise<IGetItemResponse> {
  try {
    const item = await this.itemModel.findById(id).exec();  // Query tenant-scoped DB
    return {
      success: true,
      data: item,
      message: formatMessage(Messages.success.fetchById, { entity: 'Item' }),
    };
  } catch (error) {
    return { success: false, error, message: formatMessage(Messages.error.generic) };
  }
}
```

**3. Response format** (standardized across the project):
```typescript
// Success response
{ success: true, data: { _id: "...", name: "Widget" }, message: "Item fetched" }

// Error response
{ success: false, error: {...}, message: "Something went wrong" }

// Paginated response (adds metaData)
{ success: true, data: [...], message: "...", metaData: {
    totalCount: 150, totalPages: 8,
    hasPreviousPage: true, hasNextPage: true,
    links: { current: "...", first: "...", prev: "...", next: "...", last: "..." }
  }
}
```

### Visual Flow
```
HTTP Request                  Controller                  Service                  Database
     |                            |                          |                        |
     | GET /items/abc123          |                          |                        |
     |--------------------------->|                          |                        |
     |                            | findItemById("abc123")  |                        |
     |                            |------------------------->|                        |
     |                            |                          | itemModel.findById()   |
     |                            |                          |----------------------->|
     |                            |                          |     { _id, name, ... } |
     |                            |                          |<-----------------------|
     |                            |   { success, data, msg } |                        |
     |                            |<-------------------------|                        |
     |   200 { success, data }    |                          |                        |
     |<---------------------------|                          |                        |
```

### Why Services Return `{ success, data, message }` (Not Throw Errors)
In your project, services return result objects instead of throwing exceptions. This is a design choice:
```typescript
// Your project's pattern - return result objects
return { success: false, error, message: 'Item not found' };

// Alternative pattern - throw exceptions (NOT used in your project)
throw new NotFoundException('Item not found');
```
The controller checks `success` and sets the HTTP status code accordingly.

### Common Mistakes
- **Calling one controller from another**: Controllers should never call each other. Use services instead
- **Doing DB queries in controllers**: Always delegate to the service
- **Not returning consistent response format**: Always use `formatResponse()` / `formatErrorResponse()`

---

## 6. Bearer Token Authentication

### Simple Definition
A bearer token is like a VIP wristband at a concert. Once you get it (by logging in), you show it with every request. The server checks the wristband is valid and lets you in. "Bearer" literally means "whoever bears (carries) this token gets access."

### Why It Exists
HTTP is stateless - each request is independent. The server doesn't remember you between requests. The token solves this by carrying your identity with every request.

### How It Works Conceptually
```
Step 1: Login
  Client sends credentials to Keycloak (identity provider)
  Keycloak returns: { access_token: "eyJhbGci...", refresh_token: "eyRlZnJl..." }

Step 2: Every subsequent request
  Client sends: Authorization: Bearer eyJhbGci...
  Server validates the token and extracts user info

Step 3: Token structure (JWT = JSON Web Token)
  Header:  { "alg": "RS256", "typ": "JWT" }
  Payload: { "sub": "user-id", "email": "john@co.com", "selected_tenant": "company_abc", "roles": ["admin"] }
  Signature: cryptographic proof that Keycloak created this token
```

### How It Appears in Your Project

**Keycloak Middleware** (`src/middlewares/keycloak.middleware.ts`):
```typescript
@Injectable()
export class KeycloakMiddleware implements NestMiddleware {
  private readonly client: jwksClient.JwksClient;

  constructor(private authService: AuthService) {
    // JWKS client fetches Keycloak's public keys to verify token signatures
    this.client = jwksClient({
      jwksUri: process.env.KEYCLOAK_JWKS_URI,  // e.g. https://keycloak.daic.ai/realms/main/protocol/openid-connect/certs
    });
  }

  async use(req: CustomRequest, res: Response, next: NextFunction) {
    // Extract token from "Bearer eyJhbGci..." header
    const token = this.extractTokenFromHeader(req.headers.authorization);
    // Validate signature using Keycloak's public key (RS256 algorithm)
    // Extract roles and attach to request
    next();
  }

  private extractTokenFromHeader(t: string): string | undefined {
    const [type, token] = t?.split(' ') ?? [];
    return type === 'Bearer' ? token : undefined;  // Only accept "Bearer" scheme
  }
}
```

**Token verification** (`src/auth/auth.service.ts`):
```typescript
async verifyAndDecodeToken(token: string, key: string) {
  const verify = await jwt.verify(token, key, {
    algorithms: ['RS256'],                         // Algorithm must match Keycloak
    audience: process.env.KEYCLOAK_AUDIENCE,        // Who the token is for
    issuer: process.env.KEYCLOAK_ISSUER,            // Who created the token (Keycloak URL)
    maxAge: process.env.KEYCLOAK_JWT_MAX_AGE,       // Token expiration
  });
  return verify;
}
```

**Role checking** (`src/guards/roles.guard.ts`):
```typescript
@Injectable()
export class RolesGuard implements CanActivate {
  async canActivate(context: ExecutionContext): Promise<boolean> {
    const roles = this.reflector.get<string[]>('roles', context.getHandler());  // ['admin', 'installer']
    const decodedToken = parseJwt(request.headers.authorization);
    const selectedTenant = JSON.parse(decodedToken.selected_tenant);
    const userRoles = selectedTenant.roles;  // e.g. ['admin', 'viewer']

    const hasValidRole = userRoles.some(role => roles.includes(role));
    if (!hasValidRole) throw new ForbiddenException('Forbidden error!');
    return true;
  }
}
```

### Frontend Analogy
```typescript
// Frontend: You already do this! In axios interceptors:
axios.interceptors.request.use(config => {
  config.headers.Authorization = `Bearer ${localStorage.getItem('token')}`;
  return config;
});

// Backend: The middleware receives and validates what your interceptor sends
```

### Common Mistakes
- **Storing tokens insecurely**: Never put JWTs in localStorage for production (XSS risk). Use HTTP-only cookies
- **Not checking token expiration**: Tokens expire. Frontend must handle 401 responses
- **Trusting the token payload without verifying signature**: Anyone can create a JWT; only Keycloak's signature proves it's authentic

---

## 7. Refresh Token

### Simple Definition
Access tokens expire quickly (minutes-hours) for security. A refresh token is a "get a new access token" coupon that lasts longer (days-weeks). Instead of forcing the user to log in again, the frontend silently uses the refresh token to get a new access token.

### Why It Exists
If access tokens lasted forever, a stolen token would give permanent access. Short-lived access tokens + refresh tokens balance security with user experience.

### How It Works Conceptually
```
Timeline:
  0 min   - User logs in, gets access_token (15 min) + refresh_token (7 days)
  14 min  - Access token about to expire
  15 min  - Frontend calls /auth/refresh-token with the refresh_token
          - Backend asks Keycloak for new tokens
          - Backend returns new access_token + sets new refresh_token in cookie
  30 min  - Process repeats
  7 days  - Refresh token expires, user must log in again
```

### How It Appears in Your Project

**Auth Controller** (`src/auth/auth.controller.ts`):
```typescript
@Controller('auth')
export class AuthController {
  @Get('refresh-token')
  async refreshTokenEndpoint(@Req() request: Request, @Res() response: Response) {
    // 1. Read refresh token from HTTP-only cookie (not accessible by JavaScript)
    const refreshToken = request.cookies.refreshToken;
    if (!refreshToken) throw new Error('No Refresh Token found');

    // 2. Call Keycloak to exchange refresh token for new tokens
    const tokens = await this.authService.refreshToken(refreshToken);

    // 3. Set new refresh token as HTTP-only cookie
    const cookie = serialize('refreshToken', tokens.refresh_token, {
      httpOnly: true,       // JavaScript can't read this cookie (XSS protection)
      secure: true,         // Only sent over HTTPS
      sameSite: 'none',     // Allows cross-origin requests
      path: '/',
      domain: '.daic.ai',
    });
    response.setHeader('Set-Cookie', cookie);

    // 4. Return new access token in response body
    return response.json({ accessToken: tokens.access_token });
  }
}
```

**Auth Service** (`src/auth/auth.service.ts`):
```typescript
async refreshToken(refreshToken: string) {
  // 1. Get client secret from Keycloak admin API
  const client_secret = await this.getClientSecret(process.env.KEYCLOAK_REALM);

  // 2. Call Keycloak's token endpoint with grant_type=refresh_token
  const data = qs.stringify({
    grant_type: 'refresh_token',
    client_id: process.env.KEYCLOAK_CLIENT_ID,
    refresh_token: refreshToken,
    client_secret: client_secret.data,
  });

  const response = await axios.post(
    `${process.env.KEYCLOAK_SERVER_URL}/realms/${realm}/protocol/openid-connect/token`,
    data,
    { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } }
  );

  return response.data;  // { access_token, refresh_token, expires_in, ... }
}
```

### Frontend Analogy
```typescript
// You'd call this from an axios interceptor when you get a 401:
axios.interceptors.response.use(null, async (error) => {
  if (error.response?.status === 401) {
    const { data } = await axios.get('/auth/refresh-token');  // Cookie sent automatically
    localStorage.setItem('accessToken', data.accessToken);
    error.config.headers.Authorization = `Bearer ${data.accessToken}`;
    return axios(error.config);  // Retry the failed request
  }
  return Promise.reject(error);
});
```

### Common Mistakes
- **Storing refresh token in localStorage**: This project correctly uses HTTP-only cookies. Never expose refresh tokens to JavaScript
- **Not handling refresh failure**: If refresh fails (expired), redirect to login
- **Calling refresh in a loop**: If refresh fails, don't retry endlessly. One retry, then redirect to login

---

## 8. Gateway in NestJS

### Simple Definition
A Gateway is the NestJS equivalent of a controller, but for **WebSocket connections** instead of HTTP requests. Just as `@Controller()` handles HTTP, `@WebSocketGateway()` handles real-time bidirectional communication.

### Why It Exists
HTTP is request-response: the client asks, the server answers. But for live tracking dashboards, you need the server to **push** updates to the client whenever data changes, without the client asking. WebSocket gateways enable this.

### How It Works Conceptually
```
HTTP Controller:    Client asks -----> Server responds  (one-way, then done)
WebSocket Gateway:  Client <=====> Server  (persistent connection, both can send anytime)
```

### How It Appears in Your Project

**Socket Gateway** (`src/utils/socket/socket.gateway.ts`):
```typescript
@WebSocketGateway({ cors: true })
export class SocketGateway implements OnGatewayConnection {
  @WebSocketServer() wss: Server;  // The Socket.IO server instance

  // Called when a client connects
  async handleConnection(client: any) {
    const tenantId = client.tenantId;
    client.join(tenantId!);  // Client joins a room named after their tenant
    console.log(`Client joined tenant ${tenantId}`);
  }

  // Listen for 'sendMessage' events from clients
  @SubscribeMessage('sendMessage')
  async sendMessage(client: any, payload: string) {
    const response = await this.getDataByFloorPlanId(payload, client);
    this.wss.emit(SocketEvents.Receive_Message, response);
  }

  // Push real-time data to a specific tenant's clients
  sendAssetTagRealTimeData(data: any) {
    const { tenantId, ...payload } = data;
    this.wss.to(tenantId).emit(SocketEvents.Asset_Tag_Real_Time_Data, payload);
    // Only clients in that tenant's room receive this
  }
}
```

**Events Gateway** (`src/common/test-socket.gateway.ts`):
```typescript
@UseInterceptors(RedisPropagatorInterceptor)  // Propagates events across server instances
@WebSocketGateway({ cors: { origin: whitelistedDomains, credentials: true } })
export class EventsGateway {
  @WebSocketServer() server: Server;

  @SubscribeMessage('newNotification')
  handleNewNotification(client: Socket, payload: any): void {
    this.server.emit('newNotification', payload);  // Broadcast to ALL connected clients
  }
}
```

### Key Decorators
| Decorator | Purpose |
|---|---|
| `@WebSocketGateway()` | Marks class as WebSocket handler (like `@Controller`) |
| `@WebSocketServer()` | Injects the Socket.IO Server instance |
| `@SubscribeMessage('eventName')` | Listens for specific client events (like `@Get()`) |
| `OnGatewayConnection` | Interface for handling new connections |

### Frontend Analogy
```typescript
// Frontend connects to this gateway:
const socket = io('http://localhost:5000', {
  auth: { tenantId: 'company_abc', token: 'Bearer eyJ...' }
});

socket.on('Asset_Tag_Real_Time_Data', (data) => {
  // Live update: an asset tag moved!
  updateMap(data);  // Re-render React component
});

socket.emit('sendMessage', { floorplanId: '123' });  // Ask for data
```

### Common Mistakes
- **Not joining rooms**: Without `client.join(tenantId)`, `wss.to(tenantId).emit()` sends to nobody
- **Not handling disconnection cleanup**: Memory leaks if you track sockets but don't clean up on disconnect
- **Forgetting CORS**: WebSocket connections also need CORS configuration

---

## 9. Socket.IO

### Simple Definition
Socket.IO is a library that makes WebSockets easy. It handles the messy parts: automatic reconnection, fallback to HTTP polling if WebSockets fail, rooms (groups of connections), and broadcasting.

### Why It Exists
Raw WebSockets are low-level. Socket.IO adds:
- **Auto-reconnection**: If the network drops, it reconnects automatically
- **Rooms**: Group connections by tenant, page, or feature
- **Fallback**: If WebSockets are blocked (some corporate firewalls), it falls back to HTTP long-polling
- **Acknowledgments**: Confirm the other side received the message

### How It's Used in Your Project
```
1. ASSET TRACKING (main use case):
   IoT device sends MQTT message
     → Backend processes it
     → Backend calls: wss.to(tenantId).emit('Asset_Tag_Real_Time_Data', newPosition)
     → All connected clients for that tenant see the tag move on their dashboard

2. LOCATE PAGE (search):
   Client emits: socket.emit('getLocateData', { searchText: 'forklift', entityTypes: [...] })
     → Gateway queries 24 entity types in parallel
     → Gateway emits: socket.emit('Locate_Data_Response', results)

3. NOTIFICATIONS:
   Backend emits: server.emit('newNotification', { message: 'Tag entered restricted zone' })
     → All connected clients show the notification
```

### Room Architecture (Tenant Isolation)
```
Socket.IO Server
  ├── Room: "company_abc"
  │     ├── Client 1 (John's browser)
  │     ├── Client 2 (Jane's browser)
  │     └── Client 3 (Admin tablet)
  │
  ├── Room: "company_xyz"
  │     ├── Client 4 (Bob's browser)
  │     └── Client 5 (Alice's phone)
  │
  └── wss.to("company_abc").emit(...)  → Only clients 1, 2, 3 receive this
```

### Common Mistakes
- **Emitting to all when you should emit to a room**: `server.emit()` goes to everyone. Use `server.to(room).emit()` for tenant isolation
- **Not versioning**: Frontend Socket.IO version must match backend version
- **Overusing WebSockets**: For simple request-response, HTTP is fine. Use WebSockets only for real-time push updates

---

## 10. Pub-Sub Model

### Simple Definition
Pub-Sub (Publish-Subscribe) is a messaging pattern where senders (publishers) don't send messages directly to receivers. Instead, they publish messages to a **channel**, and anyone who has subscribed to that channel receives the message. Like a radio station: the DJ broadcasts, listeners tune in.

### Why It Exists
It solves two problems:
1. **Decoupling**: The publisher doesn't need to know who's listening. You can add/remove subscribers without changing the publisher
2. **Scaling**: When you have multiple server instances, Pub-Sub (via Redis) ensures all instances stay in sync

### How It Works Conceptually
```
Without Pub-Sub (Direct):
  ServiceA → ServiceB  (tight coupling, ServiceA must know about ServiceB)

With Pub-Sub:
  ServiceA → publishes to channel "item_updated"
  ServiceB → subscribed to "item_updated" → receives automatically
  ServiceC → subscribed to "item_updated" → also receives automatically
  (ServiceA doesn't know or care about B and C)
```

### How It Appears in Your Project

**Redis Pub-Sub for cross-instance Socket.IO** (`src/common/redis-propagator/redis-propagator.service.ts`):
```typescript
@Injectable()
export class RedisPropagatorService {
  constructor(
    private readonly socketStateService: SocketStateService,
    private readonly redisService: RedisService,
  ) {
    // Subscribe to Redis channels at startup
    this.redisService.fromEvent(REDIS_SOCKET_EVENT_SEND_NAME)
      .pipe(tap(this.consumeSendEvent))
      .subscribe();

    this.redisService.fromEvent(REDIS_SOCKET_EVENT_EMIT_ALL_NAME)
      .pipe(tap(this.consumeEmitToAllEvent))
      .subscribe();
  }

  // PUBLISH: Send event to Redis channel
  public propagateEvent(eventInfo: RedisSocketEventSendDTO): boolean {
    this.redisService.publish(REDIS_SOCKET_EVENT_SEND_NAME, eventInfo);
    return true;
  }

  // SUBSCRIBE HANDLER: When event arrives from Redis, emit to local sockets
  private consumeSendEvent = (eventInfo: RedisSocketEventSendDTO): void => {
    const { userId, event, data, socketId } = eventInfo;
    this.socketStateService.get(userId)
      .filter(socket => socket.id !== socketId)  // Don't send back to originator
      .forEach(socket => socket.emit(event, data));
  };
}
```

**Redis Service - The Pub/Sub plumbing** (`src/common/redis/redis.service.ts`):
```typescript
@Injectable()
export class RedisService {
  constructor(
    @Inject(REDIS_SUBSCRIBER_CLIENT) private readonly redisSubscriberClient: RedisClient,
    @Inject(REDIS_PUBLISHER_CLIENT) private readonly redisPublisherClient: RedisClient,
  ) {}

  // Subscribe to a channel, returns an Observable stream of messages
  public fromEvent<T>(eventName: string): Observable<T> {
    this.redisSubscriberClient.subscribe(eventName);
    return new Observable(observer => {
      this.redisSubscriberClient.on('message', (channel, message) => {
        observer.next({ channel, message });
      });
    }).pipe(
      filter(({ channel }) => channel === eventName),
      map(({ message }) => JSON.parse(message)),
    );
  }

  // Publish a message to a channel
  public async publish(channel: string, value: unknown): Promise<number> {
    return this.redisPublisherClient.publish(channel, JSON.stringify(value));
  }
}
```

### Why Redis Pub-Sub Is Needed Here
```
Problem: 3 backend instances behind a load balancer
  Instance 1 has clients A, B connected via Socket.IO
  Instance 2 has clients C, D connected via Socket.IO
  Instance 3 has clients E, F connected via Socket.IO

  Instance 1 processes an MQTT event and needs to notify ALL clients.
  But Instance 1 only knows about A and B!

Solution: Redis Pub-Sub
  Instance 1 publishes event to Redis channel
  Instances 2 and 3 are subscribed to the same channel
  All instances receive the event and emit to their local clients
  Result: All 6 clients (A-F) get the update
```

### Frontend Analogy
Pub-Sub is like browser `CustomEvent`:
```typescript
// Publisher
window.dispatchEvent(new CustomEvent('cart-updated', { detail: { items: 3 } }));

// Subscriber (could be in a completely different component)
window.addEventListener('cart-updated', (e) => updateBadge(e.detail.items));
```

### Common Mistakes
- **Not unsubscribing**: Forgetting to unsubscribe from channels causes memory leaks
- **Publishing too frequently**: Redis Pub-Sub is fast but not free. Don't publish on every mouse move
- **Assuming order**: Messages may arrive out of order under load

---

## 11. Topics in Pub-Sub

### Simple Definition
A topic is the **channel name** in Pub-Sub. It's a string label that categorizes messages. Publishers send to a topic, subscribers listen to a topic. Only matching topics connect publisher to subscriber.

### Why It Exists
Without topics, every subscriber would receive every message. Topics let you filter: "I only care about order updates, not user notifications."

### How It Works Conceptually
```
Topic: "SOCKET_EVENT_SEND"
  Publisher: RedisPropagatorService.propagateEvent()
  Subscribers: All server instances listening for this topic

Topic: "SOCKET_EVENT_EMIT_ALL"
  Publisher: RedisPropagatorService.emitToAll()
  Subscribers: All server instances → broadcast to all sockets

Topic: "EXPORT_ITEM_DATA"
  Publisher: ItemsController (emits via EventEmitter)
  Subscriber: Export handler service (generates CSV)
```

### How Topics Appear in Your Project

**Redis Pub-Sub channels** (from `redis-propagator.constants.ts`):
```typescript
REDIS_SOCKET_EVENT_SEND_NAME          // Send to specific user
REDIS_SOCKET_EVENT_EMIT_ALL_NAME      // Broadcast to all
REDIS_SOCKET_EVENT_EMIT_AUTHENTICATED_NAME  // Authenticated users only
```

**NestJS EventEmitter topics** (from `src/common/nestEvents.ts`):
```typescript
export enum ENestEventTopics {
  // Bulk upload chunks
  ASSET_TAG_START_BU_CHUNK_SET = 'ASSET_TAG_START_BU_CHUNK_SET',
  ITEM_START_BU_CHUNK_SET = 'ITEM_START_BU_CHUNK_SET',

  // Data export
  EXPORT_ASSET_TAG_DATA = 'EXPORT_ASSET_TAG_DATA',
  EXPORT_ITEM_DATA = 'EXPORT_ITEM_DATA',

  // Real-time
  SEND_ASSET_TAG_REAL_TIME_DATA = 'SEND_ASSET_TAG_REAL_TIME_DATA',

  // IoT integrations
  WILLIOT_PAYLOAD_TO_RULES = 'WILLIOT_PAYLOAD_TO_RULES',
  STORE_DATA_TO_ASSET_TAG_MOVEMENT = 'STORE_DATA_TO_ASSET_TAG_MOVEMENT',

  // Configuration
  UPDATE_BRIDGE_CONFIG = 'UPDATE_BRIDGE_CONFIG',
  BULK_ARCHIVE_LOCATION_DATA = 'BULK_ARCHIVE_LOCATION_DATA',
  // ... 62 total topics
}
```

**MQTT Topics** (device communication):
```
$share/{CLIENT}/{CLIENT}/{tenantId}/publish           → General device events
$share/{CLIENT}/{CLIENT}/{tenantId}/communicator/publish/{guid}  → Keyence scanner
$share/{CLIENT}/{CLIENT}/{tenantId}/sap               → SAP integration
$share/{CLIENT}/{CLIENT}/compute-node/{macAddress}     → Edge compute devices
```

### Two Kinds of Topics in Your Project
| Kind | Technology | Example | Used For |
|---|---|---|---|
| Internal events | NestJS EventEmitter2 | `EXPORT_ITEM_DATA` | Module-to-module communication within one server |
| Distributed events | Redis Pub-Sub | `REDIS_SOCKET_EVENT_SEND` | Cross-instance communication |
| Device events | AWS MQTT | `{tenantId}/publish` | IoT device-to-server communication |

### Common Mistakes
- **Too broad topics**: A topic like `"data"` receives everything. Be specific: `"EXPORT_ITEM_DATA"`
- **Too narrow topics**: Creating a topic per user creates thousands of channels. Use rooms instead
- **Confusing EventEmitter topics with Redis topics**: EventEmitter is in-process only (one server). Redis Pub-Sub crosses server instances

---

## 12. AWS MQTT

### Simple Definition
MQTT (Message Queuing Telemetry Transport) is a lightweight messaging protocol designed for IoT devices. AWS IoT Core hosts an MQTT broker in the cloud. Your devices (scanners, tags, compute nodes) send data to AWS, and your backend subscribes to receive that data.

### Why It Exists
HTTP is too heavy for IoT devices that:
- Have limited battery/CPU/memory
- Send small messages frequently (every few seconds)
- Need reliable delivery over unreliable networks
- Need the server to push commands TO devices (not just receive)

MQTT solves all of this with minimal overhead.

### How It Works Conceptually
```
IoT Devices                     AWS IoT Core (Broker)                Your Backend

Scanner ─────publish──────>  Topic: company_abc/publish  ──subscribe──> aws-mqtt-connection-handler
                                                                            |
ComputeNode ─publish──────>  Topic: compute-node/AA:BB   ──subscribe──> handleIncomingMessage()
                                                                            |
                             <──────publish──────────────────────────── publish(ack)
                             (Backend sends acknowledgment)
```

### How It Appears in Your Project

**Connection Setup** (`src/models/aws-mqtt/aws-mqtt-connection-handler.ts`):
```typescript
@Injectable()
export class AWSMQTTConnectionService implements OnModuleInit {
  private mqttClient: mqtt.MqttClient;

  private async initializeMqttClient() {
    // 1. Load TLS certificates (mutual TLS authentication with AWS)
    const clientOptions = {
      key: Buffer.from(process.env.MQTT_AWS_KEY, 'base64').toString('ascii'),
      cert: Buffer.from(process.env.MQTT_AWS_CERT, 'base64').toString('ascii'),
      ca: [Buffer.from(process.env.MQTT_AWS_CA, 'base64').toString('ascii')],
      protocolVersion: 5,  // MQTT v5
      clientId: `${process.env.MQTT_AWS_CLIENT}-${Date.now()}`,  // Unique per instance
    };

    // 2. Connect with retry logic (up to 5 attempts with exponential backoff)
    this.mqttClient = await this.connectWithRetry(connectUrl, clientOptions, 5, 1000);

    // 3. Subscribe to topics
    this.handleSuccessfulConnect();
  }

  // Extract tenant ID from topic string
  private extractTenantIdFromTopic(topic: string): string | null {
    // Pattern: $share/{client}/{client}/{tenantId}/publish
    const match = topic.match(new RegExp(`${client}/([^/]+)/.+`));
    return match?.[1] || null;
  }
}
```

**Message Routing** (`aws-mqtt.service.ts`):
```typescript
@Injectable({ scope: Scope.REQUEST })
export class AWSMQTTService {
  // Routes incoming MQTT payloads by eventType
  async handlePayload(payload: any) {
    switch (payload.eventType) {
      case 'create':  await this.create(payload.body);  break;  // Create asset tag/item
      case 'read':    await this.read(payload.body);    break;  // Lookup tag/item
      case 'update':  await this.update(payload.body);  break;  // Update tag/item
      case 'delete':  await this.delete(payload.body);  break;  // Soft delete
      case 'notification':                                       // Just acknowledge
        await this.publish(JSON.stringify({ eventType: 'ack', body: { ... } }));
        break;
    }
  }
}
```

### Full MQTT Message Lifecycle
```
1. Barcode scanner scans a tag
2. Scanner sends MQTT message to topic: "client/company_abc/publish"
   Payload: { eventType: "read", body: { assetTagId: "TAG-001" } }

3. aws-mqtt-connection-handler receives message
4. Extracts tenantId = "company_abc" from topic
5. Routes to AWSMQTTService.handlePayload()
6. handlePayload sees eventType = "read", calls this.read()
7. read() queries tenant_company_abc.assettags for TAG-001
8. Publishes MQTT acknowledgment back to device:
   { eventType: "ack", body: { success: true, data: { tagId: "TAG-001", name: "Forklift" } } }
9. Optionally emits Socket.IO event for live dashboard update
```

### Common Mistakes
- **Not handling reconnection**: Network drops are common in IoT. Always implement reconnection with backoff (your project does this correctly)
- **Ignoring QoS levels**: QoS 0 = fire-and-forget, QoS 1 = at-least-once (your project uses QoS 1), QoS 2 = exactly-once (expensive)
- **Large payloads**: MQTT is designed for small messages. Don't send megabytes

---

## 13. MongoDB Aggregation Pipeline

### Simple Definition
An aggregation pipeline is a sequence of data processing steps (stages) that transform your MongoDB documents, one step at a time. Like an assembly line: raw data goes in, each stage modifies it, and processed results come out.

### Why It Exists
Simple `.find()` queries can't do complex operations like:
- Joining data from multiple collections
- Grouping and counting
- Computing averages/sums
- Reshaping documents

Aggregation pipelines handle all of this inside the database (much faster than doing it in JavaScript).

### How It Works Conceptually
```
Input Documents → Stage 1 → Stage 2 → Stage 3 → Output
                  $match     $lookup    $group
                  (filter)   (join)     (aggregate)
```

### Common Stages Explained
| Stage | Purpose | SQL Equivalent |
|---|---|---|
| `$match` | Filter documents | `WHERE` |
| `$lookup` | Join another collection | `LEFT JOIN` |
| `$unwind` | Flatten arrays | `LATERAL` / explode |
| `$group` | Group and aggregate | `GROUP BY` |
| `$project` | Select/reshape fields | `SELECT a, b, c` |
| `$sort` | Order results | `ORDER BY` |
| `$skip` / `$limit` | Pagination | `OFFSET` / `LIMIT` |
| `$replaceRoot` | Change document root | Subquery as source |

### How It Appears in Your Project

**Example 1: Asset Tags with Item Lookup** (`asset-tags.service.ts`):
```typescript
// "Get all asset tags WITH their linked items" (like a SQL JOIN)
const assetTagData = await this.assetTagModel.aggregate([
  // Stage 1: JOIN asset-tags with asset-tags-items (link table)
  {
    $lookup: {
      from: 'asset-tags-items',          // Collection to join
      localField: '_id',                 // AssetTag._id
      foreignField: 'assetTagId',        // AssetTagsItems.assetTagId
      as: 'assetTagsItems',             // Output array field name
    },
  },
  // Stage 2: JOIN with items collection (via the link table)
  {
    $lookup: {
      from: 'items',
      localField: 'assetTagsItems.itemId',  // From the previous lookup
      foreignField: '_id',
      as: 'item',
    },
  },
  // Result: Each asset tag now has an 'item' array with its linked items
]);
```

**Example 2: Get Latest Deleted History Per Tag** (`asset-tags.service.ts`):
```typescript
const histories = await this.assetTagHistoriesModel.aggregate([
  { $match: whereClause },                        // Stage 1: Filter
  { $sort: { tagId: 1, deletedAt: -1 } },         // Stage 2: Sort (newest first)
  {
    $group: {                                       // Stage 3: Group by tagId
      _id: '$tagId',                               //   Group key
      doc: { $first: '$$ROOT' },                   //   Take first (most recent) document
    },
  },
  { $replaceRoot: { newRoot: '$doc' } },           // Stage 4: Unwrap the grouped doc
]);
// Result: One document per tag, showing the most recent deletion
```

**Example 3: Time-Based History Aggregation** (`history.service.ts`):
```typescript
const response = await this.locationHistoryModel.aggregate([
  { $match: { updatedAt: { $gte: startDate, $lte: endDate } } },   // Filter by date range
  {
    $project: {                                                      // Extract time parts
      year: { $year: '$updatedAt' },
      month: { $month: '$updatedAt' },
      day: { $dayOfMonth: '$updatedAt' },
      hour: { $hour: '$updatedAt' },
      data: 1,
      updatedAt: 1,
    },
  },
  {
    $group: {                                                        // Group by time period
      _id: { year: '$year', month: '$month', day: '$day' },
      count: { $sum: 1 },
      firstUpdatedAt: { $first: '$updatedAt' },
    },
  },
  { $sort: { firstUpdatedAt: -1 } },                                // Most recent first
  { $skip: (page - 1) * pageSize },                                 // Pagination
  { $limit: pageSize },
]);
```

### Visual Pipeline (Example 1)
```
assettags collection              asset-tags-items               items
┌─────────────────┐              ┌──────────────────┐          ┌──────────────┐
│ _id: "tag1"     │  $lookup     │ assetTagId: "tag1"│  $lookup │ _id: "item1" │
│ tagId: "TAG-001"│──────────>   │ itemId: "item1"   │──────>   │ name: "Widget"│
│ type: "BLE"     │              │                    │          │ qty: 5       │
└─────────────────┘              └──────────────────┘          └──────────────┘

Result:
┌─────────────────────────────────────────────────────┐
│ _id: "tag1", tagId: "TAG-001", type: "BLE"         │
│ assetTagsItems: [{ assetTagId: "tag1", itemId: ... }]│
│ item: [{ _id: "item1", name: "Widget", qty: 5 }]   │
└─────────────────────────────────────────────────────┘
```

### Frontend Analogy
Aggregation pipelines work like array method chaining:
```typescript
// JavaScript (frontend):
const result = orders
  .filter(o => o.status === 'shipped')      // $match
  .map(o => ({ ...o, total: o.qty * o.price })) // $project
  .reduce((acc, o) => {                     // $group
    acc[o.customer] = (acc[o.customer] || 0) + o.total;
    return acc;
  }, {});

// MongoDB (backend):
db.orders.aggregate([
  { $match: { status: 'shipped' } },
  { $project: { customer: 1, total: { $multiply: ['$qty', '$price'] } } },
  { $group: { _id: '$customer', totalSpent: { $sum: '$total' } } },
]);
```

### Common Mistakes
- **Filtering after `$lookup`**: Always `$match` before `$lookup` to reduce the number of documents being joined (performance)
- **Forgetting `$unwind` after `$lookup`**: `$lookup` returns an array. If you need individual documents, use `$unwind`
- **Not creating indexes**: Aggregation on large collections without indexes is extremely slow
- **Over-engineering**: For simple queries, `.find()` is clearer and faster than an aggregation pipeline

---

## 14. How Everything Connects

### Architecture Diagram
```
                                    ┌──────────────────────┐
                                    │     KEYCLOAK         │
                                    │  (Identity Provider) │
                                    │  Issues JWT tokens   │
                                    └──────────┬───────────┘
                                               │ JWT validation
                                               │
   ┌──────────────────────────────────────────────────────────────────────┐
   │                        NestJS BACKEND (Port 5000)                    │
   │                                                                      │
   │  ┌─────────────────────────────────────────────────────────────┐     │
   │  │                    MIDDLEWARE CHAIN                          │     │
   │  │  Request → TenantsMiddleware → KeycloakMiddleware → Guards  │     │
   │  │           (extract tenant)   (validate JWT)    (check roles)│     │
   │  └──────────────────────────┬──────────────────────────────────┘     │
   │                             │                                        │
   │  ┌──────────────────────────┼──────────────────────────────────┐     │
   │  │                    CONTROLLER LAYER                         │     │
   │  │  ItemsController  AssetTagsController  LocationController   │     │
   │  │  CameraController  EslController  VehicleController  ...    │     │
   │  └──────────────────────────┬──────────────────────────────────┘     │
   │                             │ calls                                  │
   │  ┌──────────────────────────┼──────────────────────────────────┐     │
   │  │                    SERVICE LAYER                            │     │
   │  │  ItemsService  AssetTagsService  LocationService  ...      │     │
   │  │    │                  │                │                    │     │
   │  │    │ @Inject(PROVIDER.ITEM_MODEL)      │                   │     │
   │  │    │ (tenant-scoped Mongoose model)     │                   │     │
   │  └────┼──────────────────┼────────────────┼───────────────────┘     │
   │       │                  │                │                          │
   │  ┌────┼──────────────────┼────────────────┼───────────────────┐     │
   │  │    │          PROVIDER LAYER           │                   │     │
   │  │    │   TenantConnectionProvider        │                   │     │
   │  │    │   → mongoose.useDb('tenant_xxx')  │                   │     │
   │  │    │   createTenantModelProvider        │                   │     │
   │  └────┼──────────────────┼────────────────┼───────────────────┘     │
   │       │                  │                │                          │
   │  ┌────┼──────────────────┼────────────────┼───────────────────┐     │
   │  │    │     REAL-TIME + EVENT LAYER       │                   │     │
   │  │    │                                   │                   │     │
   │  │    │  SocketGateway ◄──── EventEmitter ────► ExportService │     │
   │  │    │       │              (in-process)                     │     │
   │  │    │       │                                               │     │
   │  │    │  RedisPropagatorService                               │     │
   │  │    │       │ (cross-instance via Redis Pub-Sub)            │     │
   │  └────┼───────┼──────────────────────────────────────────────┘     │
   │       │       │                                                     │
   └───────┼───────┼─────────────────────────────────────────────────────┘
           │       │
   ┌───────┼───────┼──────────────────────────────────────────────┐
   │       │       │          EXTERNAL SERVICES                    │
   │       ▼       ▼                                               │
   │  ┌────────┐ ┌──────────┐  ┌────────────────┐  ┌───────────┐ │
   │  │MongoDB │ │  Redis   │  │ AWS IoT (MQTT) │  │  AWS S3   │ │
   │  │        │ │          │  │                │  │  (files)  │ │
   │  │tenant_a│ │ pub/sub  │  │ Device topics  │  └───────────┘ │
   │  │tenant_b│ │ cache    │  │ Ack topics     │                 │
   │  │tenant_c│ │ sessions │  │                │  ┌───────────┐ │
   │  │  ...   │ │          │  │                │  │ AWS SES   │ │
   │  └────────┘ └──────────┘  └────────────────┘  │  (email)  │ │
   │                                                └───────────┘ │
   └──────────────────────────────────────────────────────────────┘
           │                          │
           │                          │
   ┌───────▼────────┐        ┌───────▼────────────┐
   │  Frontend App  │        │  IoT Devices        │
   │  (React)       │        │  - Scanners          │
   │  - HTTP API    │        │  - BLE Tags          │
   │  - Socket.IO   │        │  - Compute Nodes     │
   │  - JWT tokens  │        │  - Cameras           │
   └────────────────┘        └─────────────────────┘
```

---

## 15. Full Request Flow Example

### Scenario: User opens the Items page and clicks on an item

```
FRONTEND                          BACKEND                              DATABASE
────────                          ───────                              ────────

1. User clicks "Items" page
   │
   ├── axios.get('/items', {
   │     headers: {
   │       Authorization: 'Bearer eyJhbGci...',
   │       csrf-token: 'abc123'
   │     }
   │   })
   │
   │                              2. REQUEST ENTERS NestJS
   │                              │
   │                              ├── TenantsMiddleware
   │                              │   ├─ Decode JWT
   │                              │   ├─ Extract: tenantId = "acme_corp"
   │                              │   ├─ Verify tenant exists in master DB ──> tenants collection
   │                              │   ├─ Attach: req.tenantId = "acme_corp"
   │                              │   └─ next()
   │                              │
   │                              ├── KeycloakMiddleware
   │                              │   ├─ Extract Bearer token
   │                              │   ├─ Validate JWT signature via JWKS
   │                              │   ├─ Attach roles to request
   │                              │   └─ next()
   │                              │
   │                              ├── RolesGuard
   │                              │   ├─ Required: ['admin','manager','viewer']
   │                              │   ├─ User has: ['admin']
   │                              │   └─ ✅ Access granted
   │                              │
   │                              ├── TenantConnectionProvider
   │                              │   ├─ Read: req.tenantId = "acme_corp"
   │                              │   └─ Create: mongoose.useDb("tenant_acme_corp")
   │                              │
   │                              ├── ItemsController.getAllItems()
   │                              │   └─ Calls: this.itemsService.getAllItems()
   │                              │
   │                              ├── ItemsService.getAllItems()
   │                              │   ├─ this.itemModel.find({ deletedAt: null })
   │                              │   │       ↓
   │                              │   │   (itemModel is connected to ────> tenant_acme_corp.items
   │                              │   │    tenant_acme_corp database)      ┌──────────────────┐
   │                              │   │                                    │ { name: "Widget"} │
   │                              │   │                                    │ { name: "Gadget"} │
   │                              │   │                                    └──────────────────┘
   │                              │   ├─ Returns: { success: true, data: [...], message: "..." }
   │                              │
   │                              ├── ItemsController sends response
   │                              │   res.status(200).send(formatResponse(data, message))
   │
   ├── Response received:
   │   {
   │     success: true,
   │     data: [
   │       { _id: "...", name: "Widget", ... },
   │       { _id: "...", name: "Gadget", ... }
   │     ],
   │     message: "Items fetched successfully"
   │   }
   │
   └── React renders the items table
```

### Scenario: IoT scanner scans a tag (MQTT → DB → Socket → UI)

```
IOT DEVICE                  AWS IoT CORE                  BACKEND                      FRONTEND
──────────                  ────────────                  ───────                      ────────

1. Scanner reads BLE tag
   │
   ├── mqtt.publish(
   │     'client/acme_corp/publish',
   │     { eventType: 'read',
   │       body: { assetTagId: 'TAG-001' } }
   │   )
   │
   │                        2. Broker routes message
   │                        │
   │                        │                             3. aws-mqtt-connection-handler
   │                        │                             │  receives message
   │                        │                             │
   │                        │                             ├── extractTenantIdFromTopic()
   │                        │                             │   → tenantId = "acme_corp"
   │                        │                             │
   │                        │                             ├── JSON.parse(payload)
   │                        │                             │
   │                        │                             ├── Route: topic ends with "/publish"
   │                        │                             │   → POST /aws-mqtt/handle-payload
   │                        │                             │
   │                        │                             ├── AWSMQTTService.handlePayload()
   │                        │                             │   eventType = "read"
   │                        │                             │   → this.read({ assetTagId: 'TAG-001' })
   │                        │                             │
   │                        │                             ├── Queries: tenant_acme_corp.assettags
   │                        │                             │   .findOne({ tagId: 'TAG-001' })
   │                        │                             │   → { tagId: 'TAG-001', name: 'Forklift A' }
   │                        │                             │
   │                        │  ◄──────────────────────────├── Publishes MQTT ACK:
   │                        │     { eventType: 'ack',     │   { success: true, data: {...} }
   │  ◄─────────────────────│       body: {...} }         │
   │  Scanner receives ACK  │                             │
   │                        │                             ├── SocketGateway.sendAssetTagRealTimeData()
   │                        │                             │   wss.to('acme_corp').emit(
   │                        │                             │     'Asset_Tag_Real_Time_Data',
   │                        │                             │     { tagId: 'TAG-001', position: ... }
   │                        │                             │   )
   │                        │                             │
   │                        │                             │                            4. Socket.IO
   │                        │                             │                            │
   │                        │                             │                            ├── socket.on(
   │                        │                             │                            │  'Asset_Tag_Real_Time_Data',
   │                        │                             │                            │  (data) => {
   │                        │                             │                            │    updateTagPosition(data);
   │                        │                             │                            │  })
   │                        │                             │                            │
   │                        │                             │                            └── Map shows tag
   │                        │                             │                                moving in real-time
```

---

## 16. Quick Cheat Sheet

### If Your Task Involves... You Need to Know...

| Task | Key Files | Pattern |
|---|---|---|
| Add a new API endpoint | `src/models/{feature}/` controller + service + module | Follow items.controller.ts pattern |
| Add a new database entity | `submodules/entities-submodule/schemas/` + PROVIDER enum + module providers | Create schema → add to PROVIDER enum → createTenantModelProvider in module |
| Fix a query bug | `src/models/{feature}/{feature}.service.ts` | Check $match, $lookup, deletedAt filters |
| Debug auth/permissions | `keycloak.middleware.ts` → `tenant-middleware.ts` → `roles.guard.ts` | Check JWT, tenant extraction, @Roles decorator |
| Add real-time feature | `socket.gateway.ts` → Redis propagator | `wss.to(tenantId).emit(event, data)` |
| Handle IoT device data | `aws-mqtt-connection-handler.ts` → `aws-mqtt.service.ts` | Topic routing → eventType switch → DB operation → MQTT ack |
| Send email/SMS | `ses-handler.service.ts` / `sns-handler.service.ts` | Call service method with template |
| Export data | `nestEvents.ts` → EventEmitter → export handler | Emit event with IDs, handler generates CSV, sends via SES |
| Background processing | `nestEvents.ts` → `@OnEvent()` in services | `this.eventEmitter.emit(topic, payload)` |

### Decorator Quick Reference
| Decorator | Layer | Purpose |
|---|---|---|
| `@Controller('/path')` | Controller | Define HTTP route prefix |
| `@Get()` `@Post()` `@Put()` `@Delete()` | Controller | HTTP method handler |
| `@UseGuards(RolesGuard)` | Controller | Apply authorization |
| `@Roles('admin')` | Controller | Define required roles |
| `@Body()` `@Param()` `@Query()` | Controller | Extract request data |
| `@User()` | Controller | Extract decoded JWT user |
| `@Injectable()` | Service | Mark as injectable provider |
| `@Inject(TOKEN)` | Service | Inject by token (not type) |
| `@InjectModel(Name)` | Service | Inject Mongoose model |
| `@WebSocketGateway()` | Gateway | WebSocket handler |
| `@SubscribeMessage('event')` | Gateway | Listen for socket event |
| `@OnEvent('TOPIC')` | Service | Listen for internal event |
| `@Module({...})` | Module | Define module boundaries |

### Response Format Standard
```typescript
// Always use these helpers:
formatResponse(data, message)       // → { success: true, data, message }
formatErrorResponse(error, message) // → { success: false, error, message }
```
