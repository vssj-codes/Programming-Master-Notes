# NestJS — Stage 2: Request Lifecycle

> Covers Steps 6–10: Middleware, Guards, Pipes, Interceptors, Exception Filters.
> All examples built in the `library-tracker` project.

---

## The Pipeline — Big Picture

Every request passes through these layers in this fixed order:

```
HTTP Request
     │
     ▼
1. Middleware        → runs first, before everything
     │
     ▼
2. Guards            → decides if request is allowed in
     │
     ▼
3. Pipes             → validates and transforms incoming data
     │
     ▼
4. Controller        → your route handler runs here
     │
     ▼
5. Interceptors      → wraps around the handler (before + after)
     │
     ▼
6. Exception Filters → catches any errors thrown anywhere above
     │
     ▼
HTTP Response
```

### Airport Analogy

| Layer | Airport Analogy |
|---|---|
| Middleware | Entering the airport — everyone goes through |
| Guards | Passport control — are you allowed to fly? |
| Pipes | Baggage check — is your bag the right format/size? |
| Controller | Your actual flight — the destination |
| Interceptors | Lounge access — extra service before and after |
| Exception Filters | Emergency handling — something went wrong |

### DotAI Mapping

| Layer | DotAI Example |
|---|---|
| Middleware | `KeycloakMiddleware`, `TenantsMiddleware` |
| Guards | `RolesGuard`, `CSRFGuard` |
| Pipes | `ValidationPipe` (commented out in main.ts) |
| Interceptors | `RequestResponseTimeMiddleware` |
| Exception Filters | Global error handling |

---

## Step 6 — Middleware

### What is it?
Runs before everything. Reads/modifies the request, can stop it early or enrich it and pass it on.

### Key Rule
Always call `next()` unless you want to stop the request.

### Signature
```typescript
@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    // do something
    next(); // pass to next layer — critical!
  }
}
```

### What We Built — `LoggerMiddleware`
Logs every request with method, URL, status code and duration.

```typescript
// src/middleware/logger.middleware.ts
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    const { method, originalUrl } = req;
    const start = Date.now();

    res.on('finish', () => {
      const duration = Date.now() - start;
      const statusCode = res.statusCode;
      console.log(`[${method}] ${originalUrl} → ${statusCode} (${duration}ms)`);
    });

    next();
  }
}
```

### How to Register — in `app.module.ts`
```typescript
import { Module, NestModule, MiddlewareConsumer } from '@nestjs/common';
import { LoggerMiddleware } from './middleware/logger.middleware';

@Module({ ... })
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)
      .forRoutes('*');           // all routes
      // or: .forRoutes('books')  specific route
      // or: .exclude('health').forRoutes('*')  exclude one
  }
}
```

### Output
```
[GET] /books    → 200 (12ms)
[GET] /members  → 200 (2ms)
[POST] /books   → 201 (45ms)
```

### DotAI vs Library Tracker
| | DotAI | Library Tracker |
|---|---|---|
| Middleware | `KeycloakMiddleware`, `TenantsMiddleware` | `LoggerMiddleware` |
| Purpose | JWT validation, tenant DB switching | Request logging |
| Attaches to `req` | `req.roles`, `req.tenantId` | Nothing |
| Stops request? | Yes — if token invalid | No — always calls `next()` |

---

## Step 7 — Guards

### What is it?
Runs after middleware. Answers one question: **should this request be allowed?**
Returns `true` → proceed. Returns `false` → 403 Forbidden.

### Middleware vs Guard
| | Middleware | Guard |
|---|---|---|
| Job | Transform/enrich the request | Allow or block the request |
| Returns | Nothing (calls `next()`) | `true` or `false` |
| DotAI | `KeycloakMiddleware` attaches `req.roles` | `RolesGuard` checks `req.roles` |

### Signature
```typescript
@Injectable()
export class MyGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    // return true  → allow
    // return false → block (403)
  }
}
```

### What We Built — `ApiKeyGuard`
Blocks any request without a valid `x-api-key` header.

```typescript
// src/guards/api-key.guard.ts
import { CanActivate, ExecutionContext, Injectable } from '@nestjs/common';

@Injectable()
export class ApiKeyGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    const apiKey = request.headers['x-api-key'];
    const validApiKey = 'library-secret-123';
    return apiKey === validApiKey;
  }
}
```

### How to Apply — Three Levels

```typescript
// 1. Single method
@UseGuards(ApiKeyGuard)
@Get()
findAll() { ... }

// 2. Entire controller
@UseGuards(ApiKeyGuard)
@Controller('books')
export class BooksController { ... }

// 3. Globally (main.ts)
app.useGlobalGuards(new ApiKeyGuard());
```

### Test Results
```
No key        → 403 { "message": "Forbidden resource" }
Wrong key     → 403 { "message": "Forbidden resource" }
Correct key   → 200 + data
```

### DotAI vs Library Tracker
| | DotAI | Library Tracker |
|---|---|---|
| Guard | `RolesGuard`, `CSRFGuard` | `ApiKeyGuard` |
| Checks | User roles from JWT | `x-api-key` header |
| Applied | Per method | Per controller |

---

## Step 8 — Pipes

### What is it?
Runs after guards, before the controller. Two jobs:
1. **Validation** — check if incoming data is valid
2. **Transformation** — convert data types (string → number)

### Install Dependencies
```bash
npm install class-validator class-transformer
```

### Enable Globally in `main.ts`
```typescript
app.useGlobalPipes(
  new ValidationPipe({
    whitelist: true,            // strip unknown fields silently
    forbidNonWhitelisted: true, // throw 400 if unknown fields sent
  }),
);
```

### Add Validation Rules to DTOs

```typescript
// src/books/dto/create-book.dto.ts
import { IsString, IsNotEmpty, IsInt, Min, IsOptional } from 'class-validator';

export class CreateBookDto {
  @IsString()
  @IsNotEmpty()
  title: string;

  @IsString()
  @IsNotEmpty()
  author: string;

  @IsString()
  @IsOptional()        // field is not required
  genre: string;

  @IsInt()
  @Min(1)              // must be at least 1
  totalCopies: number;

  @IsInt()
  @Min(0)
  availableCopies: number;
}
```

### Common Validators

| Decorator | Rule |
|---|---|
| `@IsString()` | Must be a string |
| `@IsNotEmpty()` | Cannot be empty `""` |
| `@IsInt()` | Must be a whole number |
| `@IsNumber()` | Must be any number |
| `@Min(n)` | Must be at least n |
| `@Max(n)` | Must be at most n |
| `@IsOptional()` | Skip validation if field missing |
| `@IsEmail()` | Must be valid email format |
| `@IsBoolean()` | Must be boolean |

### ParseIntPipe — Transform URL Params

URL params are always strings. Use `ParseIntPipe` to convert them:

```typescript
@Get('page')
findPage(@Query('limit', ParseIntPipe) limit: number) {
  // limit is guaranteed to be a number here
}
```

### Test Results
```
Empty title        → 400 { "message": "title should not be empty" }
String for number  → 400 { "message": "totalCopies must be an integer number" }
Unknown field      → 400 { "message": "property hackerField should not exist" }
Valid data         → 201 + created document
```

### DotAI vs Library Tracker
| | DotAI | Library Tracker |
|---|---|---|
| Validation | Manual checks in service | `ValidationPipe` + `class-validator` |
| Applied | Per service method | Globally in `main.ts` |

---

## Step 9 — Interceptors

### What is it?
Wraps around the controller. Runs code **before AND after** the handler executes.
Perfect for: transforming responses, logging, caching, timing.

### Middleware vs Interceptor
| | Middleware | Interceptor |
|---|---|---|
| Runs | Before guards/pipes | Around the controller |
| Can modify response? | No | Yes |

### Signature
```typescript
@Injectable()
export class MyInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    // BEFORE — runs before controller

    return next.handle().pipe(
      map((data) => {
        // AFTER — runs after controller, data = controller's return value
        return data; // can transform this
      }),
    );
  }
}
```

### What We Built — `ResponseTransformInterceptor`
Automatically wraps every response in `{ success: true, data: ... }`.

```typescript
// src/interceptors/response-transform.interceptor.ts
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';

@Injectable()
export class ResponseTransformInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      map((data) => ({
        success: true,
        data: data,
      })),
    );
  }
}
```

### Register Globally in `main.ts`
```typescript
app.useGlobalInterceptors(new ResponseTransformInterceptor());
```

### Result — Every Response Now Looks Like
```json
{
  "success": true,
  "data": [ ... ]
}
```

### DotAI vs Library Tracker
| | DotAI | Library Tracker |
|---|---|---|
| Response wrapping | `formatResponse()` called manually in every method | `ResponseTransformInterceptor` once globally |
| Result | `{ success, data, message }` | `{ success, data }` |

---

## Step 10 — Exception Filters

### What is it?
The last safety net. Catches **any error** thrown anywhere in the app and sends a clean, consistent error response.

### The Problem Without It
Errors have inconsistent shapes — NestJS default error vs your custom shape don't match.

### Signature
```typescript
@Catch()  // no argument = catch ALL exceptions
export class MyFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    response.status(500).json({ ... });
  }
}
```

### What We Built — `HttpExceptionFilter`
Makes all errors return `{ success: false, data: null, message: "..." }`.

```typescript
// src/filters/http-exception.filter.ts
import {
  ExceptionFilter, Catch, ArgumentsHost,
  HttpException, HttpStatus,
} from '@nestjs/common';
import { Response } from 'express';

@Catch()
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();

    if (exception instanceof HttpException) {
      const status = exception.getStatus();
      const exceptionResponse = exception.getResponse() as any;

      const message =
        typeof exceptionResponse === 'object' && exceptionResponse.message
          ? Array.isArray(exceptionResponse.message)
            ? exceptionResponse.message.join(', ')
            : exceptionResponse.message
          : exception.message;

      return response.status(status).json({
        success: false,
        data: null,
        message: message,
      });
    }

    // unexpected crash
    console.error('Unexpected error:', exception);
    return response.status(HttpStatus.INTERNAL_SERVER_ERROR).json({
      success: false,
      data: null,
      message: 'Something went wrong',
    });
  }
}
```

### Register in `main.ts` — Before Pipes and Interceptors
```typescript
app.useGlobalFilters(new HttpExceptionFilter());     // first
app.useGlobalPipes(new ValidationPipe({ ... }));
app.useGlobalInterceptors(new ResponseTransformInterceptor());
```

### Test Results
```
403 from Guard      → { "success": false, "data": null, "message": "Forbidden resource" }
400 from Pipe       → { "success": false, "data": null, "message": "title should not be empty" }
404 from Service    → { "success": false, "data": null, "message": "Member not found" }
200 success         → { "success": true,  "data": [...] }
```

---

## Final `main.ts` — All Layers Applied

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { ValidationPipe } from '@nestjs/common';
import { ResponseTransformInterceptor } from './interceptors/response-transform.interceptor';
import { HttpExceptionFilter } from './filters/http-exception.filter';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  app.useGlobalFilters(new HttpExceptionFilter());          // catches all errors

  app.useGlobalPipes(new ValidationPipe({
    whitelist: true,
    forbidNonWhitelisted: true,
  }));                                                       // validates request body

  app.useGlobalInterceptors(new ResponseTransformInterceptor()); // wraps responses

  await app.listen(process.env.PORT ?? 3000);
}
bootstrap();
```

---

## Full Request Trace — All Layers Together

```
POST /books  (x-api-key: library-secret-123, valid body)
     │
     ▼
LoggerMiddleware     → logs "[POST] /books"
     │
     ▼
ApiKeyGuard          → checks x-api-key → ✅ valid → return true
     │
     ▼
ValidationPipe       → validates CreateBookDto → ✅ all fields valid
     │
     ▼
BooksController      → calls booksService.create(dto)
     │
     ▼
ResponseTransform    → wraps in { success: true, data: book }
     │
     ▼
Response: 201 { success: true, data: { _id: ..., title: ... } }

If anything fails at any point:
     │
     ▼
HttpExceptionFilter  → { success: false, data: null, message: "..." }
```

---

## Stage 2 — Complete Summary

| Step | Layer | Built | Purpose |
|---|---|---|---|
| 6 | Middleware | `LoggerMiddleware` | Logs every request |
| 7 | Guards | `ApiKeyGuard` | Blocks without valid key |
| 8 | Pipes | `ValidationPipe` + DTO decorators | Validates request body |
| 9 | Interceptors | `ResponseTransformInterceptor` | Wraps all responses |
| 10 | Exception Filters | `HttpExceptionFilter` | Consistent error shape |

---

## File Structure After Stage 2

```
src/
  middleware/
    logger.middleware.ts
  guards/
    api-key.guard.ts
  interceptors/
    response-transform.interceptor.ts
  filters/
    http-exception.filter.ts
  main.ts                   ← all layers registered here
  app.module.ts             ← middleware registered here
```

---

*Generated: 2026-03-23 | Project: library-tracker*
