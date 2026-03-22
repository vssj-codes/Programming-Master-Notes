# Complete Backend Learning Roadmap

> Everything you need to learn to independently navigate, understand, and modify this NestJS + MongoDB + IoT backend codebase — organized from foundational to advanced, mapped to real files in your project.

---

## How to Use This Roadmap

- **Levels 1–3**: Learn these first. They cover 80% of your daily work.
- **Levels 4–5**: Learn these once you're comfortable with the basics.
- **Level 6**: Specialized knowledge. Learn on-demand when tasks require it.
- Each topic shows the **real file in your project** where you can study the pattern.
- ✅ = You likely already know this from frontend work.

---

## LEVEL 1 — TypeScript & Node.js Foundations

> Things you need before touching NestJS. Many you'll already know from frontend.

### 1.1 TypeScript Essentials (used heavily throughout)
| Concept | What to Learn | Where It Appears |
|---------|--------------|------------------|
| ✅ Interfaces | Defining object shapes with `interface` | `submodules/enums-interfaces-submodule/interfaces/` |
| ✅ Enums | Named constants: `enum TagType { ZIM_LABEL = 'zim-label' }` | `submodules/enums-interfaces-submodule/enums/` |
| ✅ Generics | `Model<ItemDocument>`, `Observable<T>`, `Promise<IResponse>` | Every service file |
| ✅ Utility Types | `Partial<T>`, `Record<string, any>`, `Pick`, `Omit` | DTOs use `PartialType(CreateDto)` |
| Type Assertions | `as TagType`, `as unknown as Date` | Service files |
| Union / Intersection | `string \| null`, `Request & { tenantId?: string }` | Middleware, service types |
| Decorators (TS) | `@Injectable()`, `@Controller()` — how decorators work under the hood | All NestJS files |
| Declaration Merging | Extending `Request` interface with custom properties | `src/middlewares/keycloak.middleware.ts` |

### 1.2 Node.js Essentials
| Concept | What to Learn | Where It Appears |
|---------|--------------|------------------|
| ✅ async/await | Asynchronous programming with Promises | Every service method |
| ✅ Environment Variables | `process.env.VARIABLE_NAME` | `src/main.ts`, all configs |
| Buffer | `Buffer.from(cert, 'base64').toString('ascii')` | `aws-mqtt-connection-handler.ts` |
| process.hrtime | High-resolution timing for performance logging | `response-request.middleware.ts` |
| Event Loop | Understanding non-blocking I/O, how Node handles concurrency | Conceptual — affects everything |
| Streams & Observables | Node streams, RxJS Observables (`Observable`, `tap`, `map`, `filter`) | `redis.service.ts`, interceptors |
| Module System | CommonJS (`require`) vs ESM (`import`), how NestJS uses both | `tsconfig.json`, all source files |

### 1.3 HTTP Fundamentals
| Concept | What to Learn | Where It Appears |
|---------|--------------|------------------|
| ✅ HTTP Methods | GET (read), POST (create), PUT/PATCH (update), DELETE (remove) | Every controller |
| ✅ Status Codes | 200 OK, 201 Created, 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 500 Server Error | Controller response handling |
| ✅ Request/Response | Headers, body, query params, path params | Controller parameter decorators |
| ✅ CORS | Cross-Origin Resource Sharing, why browsers block requests | `src/main.ts` — `enableCors()` |
| HTTP-Only Cookies | Server-set cookies JavaScript can't read (for refresh tokens) | `src/auth/auth.controller.ts` |
| Content-Type | `application/json`, `multipart/form-data` (file uploads), `x-www-form-urlencoded` | Controllers, auth service |

---

## LEVEL 2 — Core NestJS Architecture

> The building blocks of every feature in the codebase.

### 2.1 NestJS Module System
| Concept | What to Learn | Your Project Example |
|---------|--------------|---------------------|
| `@Module()` decorator | How modules organize code into feature boundaries | `src/models/items/items.module.ts` |
| `imports` | Bringing in other modules' exported functionality | Every module file |
| `providers` | Registering injectable services and factories | Every module file |
| `controllers` | Registering HTTP request handlers | Every module file |
| `exports` | Making services available to other modules | `EslOapModule` exports `EslOapService` |
| `forRoot()` / `forRootAsync()` | Configuring global modules with static or async options | `ConfigModule.forRoot()`, `MongooseModule.forRootAsync()` in `app.module.ts` |
| `forFeature()` | Registering module-specific models/features | `MongooseModule.forFeature([...])` in `app.module.ts` |
| Feature Modules | One module per feature (items, locations, cameras, etc.) | `src/models/*/` — 66 modules |
| Dynamic Modules | Modules that accept configuration parameters | `ThrottlerModule.forRoot([...])` |

### 2.2 Dependency Injection (DI)
| Concept | What to Learn | Your Project Example |
|---------|--------------|---------------------|
| What is DI | Framework manages object creation and wiring | Conceptual — core of NestJS |
| `@Injectable()` | Marks a class as injectable (provider) | Every service: `@Injectable() export class ItemsService` |
| Constructor Injection | Declare dependencies in constructor params | Every service constructor |
| `@Inject(TOKEN)` | Inject by string/symbol token instead of class type | `@Inject(PROVIDER.ITEM_MODEL) private itemModel` |
| `@InjectModel()` | Inject Mongoose models registered with `forFeature()` | `@InjectModel(Tenant.name) private tenantModel` |
| Provider Tokens | String identifiers for non-class providers | `submodules/enums-interfaces-submodule/enums/providerEnum.ts` (130+ tokens) |
| Factory Providers | `useFactory` for custom creation logic | `src/common/redis/redis.providers.ts` |
| Injection Scopes | `Scope.DEFAULT` (singleton), `Scope.REQUEST` (per-request), `Scope.TRANSIENT` (per-injection) | `TenantConnectionProvider` uses `Scope.REQUEST` |
| `@Inject(REQUEST)` | Inject the HTTP request object into a request-scoped provider | `aws-mqtt.service.ts` |
| Circular Dependencies | When A depends on B and B depends on A, use `forwardRef()` | Watch for this in complex modules |

### 2.3 Controllers
| Concept | What to Learn | Your Project Example |
|---------|--------------|---------------------|
| `@Controller('/path')` | Define route prefix | `@Controller('/items')` |
| HTTP Method Decorators | `@Get()`, `@Post()`, `@Put()`, `@Patch()`, `@Delete()` | Every controller |
| Route Parameters | `@Param('id')` extracts from `/items/:id` | `getItemById(@Param('id') ItemId)` |
| Query Parameters | `@Query()` extracts from `?page=1&pageSize=20` | Pagination in list endpoints |
| Request Body | `@Body()` extracts parsed JSON body | `create(@Body() payload: CreateItemDto)` |
| Response Object | `@Res()` injects Express Response for manual control | All controller methods use `@Res()` |
| Custom Param Decorators | `createParamDecorator()` for reusable extraction logic | `src/decorators/user.decorator.ts` — `@User()` |

### 2.4 Services
| Concept | What to Learn | Your Project Example |
|---------|--------------|---------------------|
| Business Logic Layer | Services contain ALL logic — controllers don't | Every `*.service.ts` file |
| Return Pattern | `{ success: boolean, data: any, message: string }` | Standardized across all services |
| Error Handling | try/catch wrapping, returning error objects vs throwing | Every service method |
| Service-to-Service Calls | Injecting one service into another | `AssetTagsService` injects `WiliotService`, `LocationService`, etc. |
| Model Queries | `find()`, `findOne()`, `findById()`, `create()`, `findOneAndUpdate()`, `deleteOne()` | Every service |

### 2.5 Response Formatting
| Concept | What to Learn | Your Project Example |
|---------|--------------|---------------------|
| `formatResponse()` | Standard success response wrapper | `src/common/formatResponse.ts` |
| `formatErrorResponse()` | Standard error response wrapper | `src/common/formatResponse.ts` |
| Pagination Metadata | `{ totalCount, totalPages, hasPreviousPage, hasNextPage, links }` | `src/common/dto/pagination-query.dto.ts` |
| `formatMessage()` | Template-based message formatting | Used throughout controllers |

---

## LEVEL 3 — NestJS Request Pipeline & Data Layer

> How requests flow through the system, and how data is stored/retrieved.

### 3.1 Middleware
| Concept | What to Learn | Your Project Example |
|---------|--------------|---------------------|
| `NestMiddleware` interface | Class-based middleware with `use(req, res, next)` | `src/middlewares/keycloak.middleware.ts` |
| Middleware registration | `consumer.apply(Middleware).forRoutes('*')` | `app.module.ts` → `configure()` method |
| Route exclusion | `.exclude({ path: '...', method: RequestMethod.GET })` | `app.module.ts` |
| Middleware ordering | Execution order matters: tenant → auth → validation | `app.module.ts` |
| Request mutation | Adding properties to `req` object for downstream use | TenantsMiddleware adds `req.tenantId`, `req.user` |
| Timing middleware | Measuring request/response time | `src/middlewares/response-request.middleware.ts` |

### 3.2 Guards
| Concept | What to Learn | Your Project Example |
|---------|--------------|---------------------|
| `CanActivate` interface | Return `true` to allow, throw/`false` to block | `src/guards/roles.guard.ts` |
| `@UseGuards()` | Apply guards to controllers or individual routes | `@UseGuards(RolesGuard, CSRFGuard)` |
| `Reflector` | Read metadata set by decorators (like `@Roles()`) | `roles.guard.ts` — `reflector.get<string[]>('roles', ...)` |
| `ExecutionContext` | Access request, response, handler info inside guard | All guard files |
| Guard ordering | Left to right: `@UseGuards(First, Second)` — First runs first | Controller decorators |
| RolesGuard | Check JWT roles against route requirements | `src/guards/roles.guard.ts` |
| CSRFGuard | Validate CSRF token header for mutation requests | `src/guards/csrf.guard.ts` |
| SubscriptionsGuard | Check subscription/plan access | `src/guards/subscriptions.guard.ts` |
| WsThrottlerGuard | Rate limit WebSocket connections | `src/guards/webSocketThrottle.guard.ts` |

### 3.3 Pipes (Validation & Transformation)
| Concept | What to Learn | Your Project Example |
|---------|--------------|---------------------|
| `PipeTransform` interface | Validate/transform incoming data | `src/models/asset-tags/asset-tag-type.pipe.ts` |
| `ValidationPipe` | Auto-validate DTOs using class-validator decorators | Global pipe or per-route |
| Custom Pipes | Validate route params (e.g., tag type enum validation) | `AssetTagTypePipe` |
| `@UsePipes()` | Apply pipes to specific routes | Per-route validation |

### 3.4 Exception Filters
| Concept | What to Learn | Your Project Example |
|---------|--------------|---------------------|
| `ExceptionFilter` interface | Centralized error response formatting | `src/filters/all-exceptions.filter.ts` |
| `@Catch()` | Declare which exceptions to catch (or all with `@Catch()`) | `all-exceptions.filter.ts` |
| `ArgumentsHost` | Access request/response context in filter | `all-exceptions.filter.ts` |
| HttpException | Built-in exceptions: `BadRequestException`, `ForbiddenException`, `NotFoundException` | Used in guards and services |

### 3.5 Interceptors
| Concept | What to Learn | Your Project Example |
|---------|--------------|---------------------|
| `NestInterceptor` interface | Wrap handler execution (before + after logic) | `redis-propagator.interceptor.ts` |
| `CallHandler` / `next.handle()` | Execute the actual handler, get response as Observable | `redis-propagator.interceptor.ts` |
| RxJS `tap()` operator | Side effect after handler completes (logging, propagation) | Redis propagator uses `tap()` to publish to Redis |
| `@UseInterceptors()` | Apply interceptors to controllers or gateways | `EventsGateway` uses `@UseInterceptors(RedisPropagatorInterceptor)` |

### 3.6 Custom Decorators
| Concept | What to Learn | Your Project Example |
|---------|--------------|---------------------|
| `SetMetadata()` | Attach metadata to route handler (read by guards) | `src/decorators/roles.decorator.ts` — `@Roles()` |
| `createParamDecorator()` | Create custom parameter extractors | `src/decorators/user.decorator.ts` — `@User()` |
| `applyDecorators()` | Combine multiple decorators into one | `src/decorators/swagger-api-roles.decorators.ts` — `@ApiRolesAllowed()` |

### 3.7 NestJS Request Pipeline (Execution Order)
```
Request → Middleware → Guards → Interceptors (before) → Pipes → Controller → Service
    → Interceptors (after) → Exception Filters (if error) → Response
```
Learn this order by heart — it explains when each piece runs.

### 3.8 MongoDB with Mongoose
| Concept | What to Learn | Your Project Example |
|---------|--------------|---------------------|
| Schema Definition | `@Schema()`, `@Prop()`, `SchemaFactory.createForClass()` | `submodules/entities-submodule/schemas/asset-tags.schema.ts` |
| Prop Options | `type`, `default`, `enum`, `trim`, `required`, `ref` | All schema files |
| Indexes | `schema.index({ field: -1 }, { unique: true })` | `AssetTagSchema.index({ tagId: -1 })` |
| Static Methods | `schema.statics.createWithHistory = async function(...)` | All schemas have `createWithHistory`, `findByIdSafe` |
| Timestamps | `@Schema({ timestamps: true })` auto-adds `createdAt`/`updatedAt` | All schemas |
| Collection Naming | `@Schema({ collection: CollectionName.ASSET_TAGS })` | All schemas |
| Document Types | `AssetTagsDocument = AssetTags & Document` | Used as `Model<AssetTagsDocument>` |
| Lean Queries | `.lean()` returns plain JS objects (faster, no Mongoose overhead) | Used in find queries |
| Population | `.populate('fieldName')` to resolve referenced documents | Some service queries |

### 3.9 Mongoose Query Methods
| Method | Purpose | SQL Equivalent |
|--------|---------|---------------|
| `.find(filter)` | Find multiple documents | `SELECT * WHERE ...` |
| `.findOne(filter)` | Find single document | `SELECT * WHERE ... LIMIT 1` |
| `.findById(id)` | Find by `_id` | `SELECT * WHERE id = ?` |
| `.create(data)` | Insert new document | `INSERT INTO ...` |
| `.findOneAndUpdate(filter, update, opts)` | Find and update atomically | `UPDATE ... WHERE ... RETURNING *` |
| `.updateMany(filter, update)` | Update multiple documents | `UPDATE ... WHERE ...` |
| `.deleteOne(filter)` | Hard delete one document | `DELETE FROM ... WHERE ...` |
| `.countDocuments(filter)` | Count matching documents | `SELECT COUNT(*)` |
| `.aggregate([...])` | Run aggregation pipeline | Complex queries with JOINs |
| `.sort({ field: 1 or -1 })` | Sort ascending (1) or descending (-1) | `ORDER BY` |
| `.skip(n).limit(n)` | Pagination | `OFFSET ... LIMIT ...` |

### 3.10 DTOs & Validation (class-validator)
| Decorator | Purpose | Example |
|-----------|---------|---------|
| `@IsString()` | Validate string type | `@IsString() name: string` |
| `@IsNumber()` | Validate number type | `@IsNumber() maxWeight: number` |
| `@IsBoolean()` | Validate boolean type | `@IsBoolean() enabled: boolean` |
| `@IsOptional()` | Allow field to be undefined | `@IsOptional() description?: string` |
| `@IsNotEmpty()` | Require non-empty value | `@IsNotEmpty() tagId: string` |
| `@IsMongoId()` | Validate MongoDB ObjectId format | `@IsMongoId() transitTagId: Types.ObjectId` |
| `@IsObject()` | Validate object type | `@IsObject() properties: object` |
| `@IsArray()` | Validate array type | `@IsArray() ids: string[]` |
| `@IsEnum()` | Validate against enum values | `@IsEnum(SortOrder) sortBy: SortOrder` |
| `@ValidateNested()` | Validate nested objects | `@ValidateNested() properties: PropertiesDto` |
| `@Type(() => Class)` | Transform nested to class instance | `@Type(() => PropertiesDto)` |
| `PartialType(Dto)` | Make all fields optional (for update DTOs) | `class UpdateDto extends PartialType(CreateDto)` |

---

## LEVEL 4 — Authentication, Real-Time, Multi-Tenancy

> The patterns that make this project unique.

### 4.1 Authentication & Authorization
| Concept | What to Learn | Your Project Example |
|---------|--------------|---------------------|
| JWT (JSON Web Token) | Header.Payload.Signature structure, claims, expiration | Used in every authenticated request |
| RS256 Signing | Asymmetric signing — private key signs, public key verifies | Keycloak uses RS256 |
| JWKS (JSON Web Key Set) | Endpoint that publishes public keys for verification | `KeycloakMiddleware` uses `jwks-rsa` library |
| Bearer Token Scheme | `Authorization: Bearer <token>` header convention | `keycloak.middleware.ts` |
| Token Introspection | Calling identity provider to check if token is still valid | `authService.introspectToken()` |
| Refresh Tokens | Long-lived token exchanged for new access token | `src/auth/auth.controller.ts` |
| HTTP-Only Cookies | Secure cookie storage for refresh tokens | `serialize('refreshToken', ...)` with `httpOnly: true` |
| CSRF Protection | Prevent cross-site request forgery on mutations | `src/guards/csrf.guard.ts` |
| OpenID Connect (OIDC) | Identity protocol built on OAuth2 (Keycloak implements this) | `openid-client` library in `auth.service.ts` |
| Keycloak | Identity and access management server | Central auth system |
| Role-Based Access Control (RBAC) | Restrict endpoints based on user roles | `@Roles('admin')` + `RolesGuard` |

### 4.2 Multi-Tenancy
| Concept | What to Learn | Your Project Example |
|---------|--------------|---------------------|
| Tenant Isolation Strategies | Shared DB, Schema-per-tenant, **Database-per-tenant** (yours) | Your project uses database-per-tenant |
| Tenant Middleware | Extract tenant from request, validate, set context | `submodules/helpers-submodule/middlewares/tenant-middleware.ts` |
| Dynamic Database Switching | `connection.useDb('tenant_xxx')` per request | `submodules/helpers-submodule/provider/tenant-connection.provider.ts` |
| Request-Scoped Providers | `Scope.REQUEST` — new provider instance per HTTP request | `TenantConnectionProvider` |
| Tenant Model Factory | `createTenantModelProvider()` — create models bound to tenant DB | `submodules/helpers-submodule/provider/tenant-model.provider.ts` |
| Internal Service Calls | `x-internal-request` + `x-internal-secret` headers to bypass JWT | Used when services call each other internally |

### 4.3 WebSockets & Socket.IO
| Concept | What to Learn | Your Project Example |
|---------|--------------|---------------------|
| WebSocket Protocol | Full-duplex communication over a single TCP connection | Conceptual |
| Socket.IO | Library that wraps WebSockets with rooms, auto-reconnect, fallback | `socket.io` package |
| `@WebSocketGateway()` | NestJS gateway class for handling WebSocket connections | `src/utils/socket/socket.gateway.ts` |
| `@WebSocketServer()` | Inject the Socket.IO Server instance | `wss: Server` in gateways |
| `@SubscribeMessage()` | Listen for named events from clients | `@SubscribeMessage('sendMessage')` |
| `OnGatewayConnection` | Interface for handling new socket connections | `SocketGateway implements OnGatewayConnection` |
| Rooms | Group sockets by tenant/feature — `socket.join(roomId)` | `client.join(tenantId)` |
| Room-Scoped Emission | `wss.to(roomId).emit(event, data)` — only room members receive | `sendAssetTagRealTimeData()` |
| Socket Authentication | Validate JWT during WebSocket handshake | `src/common/socket-state/socket-state.adapter.ts` |
| Socket State Management | Track `Map<userId, Socket[]>` in memory | `src/common/socket-state/socket-state.service.ts` |

### 4.4 Redis
| Concept | What to Learn | Your Project Example |
|---------|--------------|---------------------|
| Redis Basics | In-memory key-value store — get/set/delete with TTL | `src/common/redis/redis.service.ts` |
| Redis Pub/Sub | Publish messages to channels, subscribers receive them | `redis.service.ts` — `publish()`, `fromEvent()` |
| Two-Client Pattern | Separate clients for publishing and subscribing | `REDIS_PUBLISHER_CLIENT`, `REDIS_SUBSCRIBER_CLIENT` |
| Cross-Instance Propagation | Use Redis Pub/Sub to sync Socket.IO events across server instances | `src/common/redis-propagator/redis-propagator.service.ts` |
| Cache with TTL | `SETEX key ttl value` — auto-expire after seconds | `redis.service.ts` — `set(key, ttl, value)` |
| ioredis Library | Redis client for Node.js with Sentinel/Cluster support | `src/common/redis/redis.providers.ts` |

### 4.5 Event-Driven Architecture
| Concept | What to Learn | Your Project Example |
|---------|--------------|---------------------|
| EventEmitter2 | NestJS event bus for decoupled module communication | `@nestjs/event-emitter` |
| `EventEmitterModule.forRoot()` | Register the event system globally | `app.module.ts` |
| Emitting Events | `this.eventEmitter.emit(topicName, payload)` | Throughout services |
| `@OnEvent(topic)` | Decorator to listen for specific events | Event handler methods in services |
| Event Topics Enum | Centralized list of all event names | `src/common/nestEvents.ts` — 62 topics |
| Fire-and-Forget | Events are async — emitter doesn't wait for handler | Used for exports, background processing |

---

## LEVEL 5 — IoT, DevOps & Advanced Patterns

> Specialized knowledge for device communication and deployment.

### 5.1 MQTT Protocol
| Concept | What to Learn | Your Project Example |
|---------|--------------|---------------------|
| MQTT Basics | Lightweight publish/subscribe messaging protocol for IoT | `src/models/aws-mqtt/` |
| Broker | Central server that routes messages (AWS IoT Core is yours) | AWS IoT Core |
| Topics (MQTT) | Hierarchical strings: `client/tenantId/publish` | Topic patterns in `aws-mqtt-connection-handler.ts` |
| QoS Levels | 0 (fire-forget), **1 (at-least-once — yours)**, 2 (exactly-once) | MQTT client config |
| Shared Subscriptions | `$share/{group}/{topic}` — load balance across subscribers | Your topic patterns use `$share/` |
| Mutual TLS | Both client and server present certificates | Certificate-based auth with AWS IoT |
| Reconnection Strategy | Exponential backoff: 1s → 2s → 4s → ... → 30s max | `aws-mqtt-connection-handler.ts` |
| MQTT v5 | Protocol version with enhanced features | `protocolVersion: 5` in connection config |
| Acknowledgment | Backend publishes ACK message back to device topic | `eventType: 'ack'` in `aws-mqtt.service.ts` |

### 5.2 MongoDB Aggregation Pipeline
| Stage | What to Learn | Your Project Example |
|-------|--------------|---------------------|
| `$match` | Filter documents (like WHERE) | Filtering by `deletedAt: null` |
| `$lookup` | Join with another collection (like LEFT JOIN) | Joining `asset-tags` with `items` |
| `$unwind` | Flatten arrays into individual documents | Unwinding lookup results |
| `$group` | Group documents and compute aggregates (`$sum`, `$first`, `$avg`) | Grouping histories by `tagId` |
| `$project` | Select/rename/compute fields | Extracting `$year`, `$month` from dates |
| `$sort` | Order results | `{ createdAt: -1 }` for newest first |
| `$skip` / `$limit` | Pagination in aggregation | Used with page calculations |
| `$replaceRoot` | Promote nested document to root level | `{ newRoot: '$doc' }` after `$group` |
| `$addFields` | Add computed fields without removing existing ones | Used for enrichment |
| `$facet` | Run multiple pipelines in parallel on same data | Used for pagination (count + data) |
| Date Operators | `$year`, `$month`, `$dayOfMonth`, `$hour`, `$isoWeek` | `history.service.ts` |

### 5.3 API Versioning
| Concept | What to Learn | Your Project Example |
|---------|--------------|---------------------|
| URI Versioning | `/v1/items`, `/v2/items` — version in URL path | `app.enableVersioning({ type: VersioningType.URI })` |
| `@Version()` decorator | Mark controller methods with version | `@Version(CURRENT_VERSION)` on specific endpoints |
| Version Constants | Centralized version identifiers | `CURRENT_VERSION` constant |

### 5.4 File Uploads
| Concept | What to Learn | Your Project Example |
|---------|--------------|---------------------|
| Multer | Express middleware for handling `multipart/form-data` | `@UseInterceptors(FileInterceptor('file', multerOptions))` |
| `@UploadedFile()` | Extract uploaded file from request | `uploadFile(@UploadedFile() file: Express.Multer.File)` |
| multer-s3 | Stream uploads directly to AWS S3 | AWS S3 integration |
| `FileInterceptor` | NestJS wrapper around multer for single file upload | `src/models/file/file.controller.ts` |

### 5.5 Swagger / OpenAPI Documentation
| Concept | What to Learn | Your Project Example |
|---------|--------------|---------------------|
| `DocumentBuilder` | Configure API docs title, description, auth scheme | `src/swagger.setup.ts` |
| `SwaggerModule.createDocument()` | Generate OpenAPI spec from decorators | `swagger.setup.ts` |
| `@ApiTags()` | Group endpoints in Swagger UI | `@ApiTags('Items')` on controllers |
| `@ApiBearerAuth()` | Document that endpoints require JWT | All controller classes |
| `@ApiResponse()` | Document response types and status codes | Individual endpoints |
| `@ApiProperty()` | Document DTO fields in Swagger | DTO classes |
| `@ApiQuery()` | Document query parameters | Pagination endpoints |
| Basic Auth Protection | Protect Swagger UI with password in production | `basicAuth()` middleware in `swagger.setup.ts` |

### 5.6 Logging
| Concept | What to Learn | Your Project Example |
|---------|--------------|---------------------|
| Winston Logger | Configurable logging with levels, formats, transports | `src/utils/logger/customLogger.ts` |
| Log Levels | `error`, `warn`, `info`, `debug` | `ERROR_TYPES.INFO`, `ERROR_TYPES.ERROR` |
| Log Format | JSON, timestamp, colorize, printf | Winston format combinators |
| NestJS Logger Integration | Replace default logger with Winston | `WinstonModule.createLogger()` in `main.ts` |
| Custom Log Functions | `logInfo()`, `logError()` wrappers | `src/services/customLoggerService.ts` |

### 5.7 Security Best Practices
| Concept | What to Learn | Your Project Example |
|---------|--------------|---------------------|
| Helmet | Security headers (HSTS, CSP, X-Frame-Options, etc.) | `app.use(helmet({...}))` in `main.ts` |
| mongo-sanitize | Prevent NoSQL injection (`{ $gt: '' }` in query params) | `app.use(mongoSanitize())` in `main.ts` |
| Rate Limiting | Throttle requests to prevent abuse | `ThrottlerModule.forRoot([{ ttl: 60000, limit: 60 }])` |
| CORS Configuration | Whitelist allowed origins | `enableCors({ origin: whitelistedDomains })` |
| Graceful Shutdown | Clean up connections on process exit | `app.enableShutdownHooks()` |

### 5.8 Lifecycle Hooks
| Hook | When It Runs | Your Project Example |
|------|-------------|---------------------|
| `OnModuleInit` | After module is initialized | `AWSMQTTConnectionService.onModuleInit()` |
| `OnModuleDestroy` | Before module is destroyed | Cleanup connections |
| `OnApplicationBootstrap` | After all modules are initialized | App-level setup |
| `OnApplicationShutdown` | During graceful shutdown | `ShutdownService` |

### 5.9 AWS Services
| Service | What to Learn | Your Project Example |
|---------|--------------|---------------------|
| AWS S3 | Object storage — file upload/download, signed URLs | `src/clients/aws.client.ts` |
| AWS SES | Email sending — templates, raw emails, attachments | `src/models/ses-handler/ses-handler.service.ts` |
| AWS SNS | Push notifications, SMS sending | `src/models/sns-handler/sns-handler.service.ts` |
| AWS Secrets Manager | Store and retrieve secrets (Keycloak client secret) | `auth.service.ts` |
| AWS IoT Core | Managed MQTT broker for device communication | `aws-mqtt-connection-handler.ts` |

---

## LEVEL 6 — DevOps & Infrastructure (Learn On-Demand)

> You probably won't need these daily, but understanding them helps.

### 6.1 Docker
| Concept | What to Learn | Your Project File |
|---------|--------------|-------------------|
| Dockerfile | Build instructions: base image, copy, install, build | `Dockerfile` |
| docker-compose | Multi-container setup: app + Redis | `docker-compose.yml` |
| Volumes | Mount local code for hot-reload in development | docker-compose volumes |
| Networks | Container-to-container communication | `backend-network` bridge |

### 6.2 Kubernetes
| Concept | What to Learn | Your Project File |
|---------|--------------|-------------------|
| Pods / Deployments | Running containers in K8s | `kubernetes/backend/templates/deployment.yaml` |
| Services / Ingress | Exposing apps to network | `kubernetes/backend/templates/service.yaml` |
| Helm Charts | Templated K8s deployments | `kubernetes/backend/Chart.yaml` |
| HPA / VPA | Auto-scaling based on CPU/memory | `kubernetes/backend/templates/` |
| Readiness/Liveness Probes | Health checks for container management | Deployment template |

### 6.3 CI/CD
| Concept | What to Learn | Your Project File |
|---------|--------------|-------------------|
| GitLab CI | Pipeline stages: check → build → test → deploy | `.gitlab-ci.yml` |
| Docker Registry | Push/pull container images | `registry.gitlab.com/seeid/...` |
| Environment-specific configs | Dev, staging, production value files | `kubernetes/backend/values-*.yaml` |

### 6.4 Kubernetes Client (Programmatic)
| Concept | What to Learn | Your Project File |
|---------|--------------|-------------------|
| Kubernetes API from code | Create/manage K8s resources programmatically | `src/models/kubernetes/kubernetes.service.ts` |
| Custom Resources | Define domain-specific K8s objects | CRDs for tenant provisioning |

---

## How These Pieces Connect — The Complete Request Pipeline

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      NestJS REQUEST PIPELINE                             │
│                                                                          │
│  HTTP Request                                                            │
│       │                                                                  │
│       ▼                                                                  │
│  ┌─────────────────┐   You'll learn this in:                            │
│  │   MIDDLEWARE     │   Level 3.1                                        │
│  │  TenantsMiddleware   → extracts tenant from JWT                      │
│  │  KeycloakMiddleware  → validates JWT signature                       │
│  │  ResponseTimeMiddleware → logs timing                                │
│  └────────┬────────┘                                                    │
│           ▼                                                              │
│  ┌─────────────────┐   Level 3.2                                        │
│  │     GUARDS      │                                                    │
│  │  RolesGuard     → checks @Roles() vs JWT roles                      │
│  │  CSRFGuard      → validates csrf-token header                       │
│  └────────┬────────┘                                                    │
│           ▼                                                              │
│  ┌─────────────────┐   Level 3.5                                        │
│  │  INTERCEPTORS   │   (before handler)                                 │
│  │  RedisPropagator → wraps WebSocket handlers                         │
│  └────────┬────────┘                                                    │
│           ▼                                                              │
│  ┌─────────────────┐   Level 3.3                                        │
│  │     PIPES       │                                                    │
│  │  ValidationPipe → validates @Body() against DTO                     │
│  │  AssetTagTypePipe → validates @Param() values                       │
│  └────────┬────────┘                                                    │
│           ▼                                                              │
│  ┌─────────────────┐   Level 2.3                                        │
│  │   CONTROLLER    │                                                    │
│  │  @Get('/:id')   → routes request to handler method                  │
│  │  @User()        → extracts decoded JWT as parameter                 │
│  │  @Param('id')   → extracts URL parameter                           │
│  └────────┬────────┘                                                    │
│           ▼                                                              │
│  ┌─────────────────┐   Level 2.4                                        │
│  │    SERVICE      │                                                    │
│  │  Business logic, DB queries, external calls                         │
│  │  @Inject(PROVIDER.MODEL) → tenant-scoped Mongoose model            │
│  └────────┬────────┘                                                    │
│           ▼                                                              │
│  ┌─────────────────┐   Level 3.8                                        │
│  │    DATABASE     │                                                    │
│  │  MongoDB: tenant_acme_corp                                          │
│  │  .find() / .aggregate() / .create()                                 │
│  └────────┬────────┘                                                    │
│           ▼                                                              │
│  ┌─────────────────┐   Level 2.5                                        │
│  │   RESPONSE      │                                                    │
│  │  formatResponse(data, message)                                      │
│  │  → { success: true, data: [...], message: "..." }                   │
│  └─────────────────┘                                                    │
│                                                                          │
│  If error at any stage:                                                  │
│  ┌─────────────────┐   Level 3.4                                        │
│  │ EXCEPTION FILTER│                                                    │
│  │  Catches thrown exceptions, formats error response                  │
│  │  → { success: false, message: "...", error: {...} }                 │
│  └─────────────────┘                                                    │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Suggested Learning Order (Week by Week)

| Week | Focus | Goal |
|------|-------|------|
| 1 | Level 1 (review) + Level 2.1–2.3 | Understand modules, DI, controllers |
| 2 | Level 2.4–2.5 + Level 3.8–3.10 | Write services, query MongoDB, validate DTOs |
| 3 | Level 3.1–3.4 | Understand middleware, guards, pipes, filters |
| 4 | Level 3.5–3.7 + Level 4.2 | Interceptors, custom decorators, multi-tenancy |
| 5 | Level 4.1 | Authentication (JWT, Keycloak, RBAC) |
| 6 | Level 4.3–4.4 | WebSockets, Socket.IO, Redis |
| 7 | Level 4.5 + Level 5.1 | Events, MQTT, IoT device flows |
| 8 | Level 5.2–5.9 | Aggregation pipelines, file uploads, Swagger, logging, security, AWS |
| 9+ | Level 6 (on-demand) | Docker, Kubernetes, CI/CD as needed |

---

## Practice Exercises (Using Your Codebase)

| # | Exercise | Concepts Practiced |
|---|----------|-------------------|
| 1 | Read `src/models/items/items.module.ts` → trace how ItemsService gets its dependencies | Modules, Providers, DI, @Inject |
| 2 | Read `src/models/items/items.controller.ts` → follow one endpoint from request to response | Controllers, Guards, Response formatting |
| 3 | Read `src/guards/roles.guard.ts` → understand how @Roles() and RolesGuard work together | Guards, Reflector, SetMetadata |
| 4 | Read `src/middlewares/keycloak.middleware.ts` → trace how JWT is validated | Middleware, JWT, JWKS |
| 5 | Read `submodules/helpers-submodule/provider/tenant-connection.provider.ts` → understand database switching | Multi-tenancy, Request scope, Factory providers |
| 6 | Read `submodules/entities-submodule/schemas/asset-tags.schema.ts` → understand schema patterns | Mongoose, Schema, Static methods, Indexes |
| 7 | Read `src/utils/socket/socket.gateway.ts` → trace how real-time data reaches the frontend | WebSocket Gateway, Rooms, Socket.IO |
| 8 | Read `src/common/redis-propagator/redis-propagator.service.ts` → understand cross-instance events | Redis Pub/Sub, Interceptors, RxJS |
| 9 | Find an `.aggregate()` call in any service → draw the pipeline stages on paper | Aggregation pipeline, $lookup, $match, $group |
| 10 | Read `src/models/aws-mqtt/aws-mqtt-connection-handler.ts` → trace an MQTT message from device to DB | MQTT, Topic routing, IoT flow |
