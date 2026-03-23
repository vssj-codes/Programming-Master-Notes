# Step 11 — DTOs & Validation (Deep Dive)

> Stage 3: Data & Config | Project: library-tracker

---

## DTO vs Schema — The Key Difference

This trips up most beginners. Both look similar but serve completely different purposes.

| | DTO | Schema |
|---|---|---|
| What it is | Shape of **incoming request** data | Shape of **database document** |
| Where used | Controller/Pipe layer | Mongoose/MongoDB layer |
| Purpose | Validate what the client sends | Define what gets stored |
| File | `dto/create-book.dto.ts` | `book.schema.ts` |
| Decorators | `class-validator` (`@IsString()` etc.) | Mongoose (`@Prop()`) |

### The Flow

```
Client sends JSON
      │
      ▼
DTO validates it      ← class-validator decorators
      │
      ▼
Service processes it
      │
      ▼
Schema saves it       ← Mongoose @Prop() decorators
      │
      ▼
MongoDB stores it
```

---

## Part 1 — DTO Transformation (`transform: true`)

Enable in `main.ts` so NestJS auto-converts types instead of rejecting them:

```typescript
app.useGlobalPipes(
  new ValidationPipe({
    whitelist: true,
    forbidNonWhitelisted: true,
    transform: true,  // "3" (string) → 3 (number) automatically
  }),
);
```

With `transform: true`, NestJS reads the TypeScript type on the DTO field and converts:
```typescript
@IsInt()
totalCopies: number;  // TypeScript says number → "3" gets converted to 3
```

---

## Part 2 — Nested DTOs

When the request body has an object inside it, validate it with a nested DTO.

### Step 1 — Create the nested DTO

```typescript
// src/books/dto/publisher.dto.ts
import { IsString, IsNotEmpty, IsOptional } from 'class-validator';

export class PublisherDto {
  @IsString()
  @IsNotEmpty()
  name: string;

  @IsString()
  @IsOptional()
  country?: string;
}
```

### Step 2 — Use it inside the parent DTO

```typescript
// src/books/dto/create-book.dto.ts
import { IsString, IsNotEmpty, IsInt, Min, IsOptional, ValidateNested } from 'class-validator';
import { Type } from 'class-transformer';
import { PublisherDto } from './publisher.dto';

export class CreateBookDto {
  @IsString()
  @IsNotEmpty()
  title: string;

  @IsString()
  @IsNotEmpty()
  author: string;

  @IsString()
  @IsOptional()
  genre?: string;

  @IsInt()
  @Min(1)
  totalCopies: number;

  @IsInt()
  @Min(0)
  availableCopies: number;

  @IsOptional()
  @ValidateNested()          // validate the nested object too
  @Type(() => PublisherDto)  // tell class-transformer what type to instantiate
  publisher?: PublisherDto;
}
```

### Key Decorators for Nested Objects

```typescript
@ValidateNested()         // tells ValidationPipe: go inside this object and validate it
@Type(() => PublisherDto) // needed by class-transformer to know what class to use
                          // WITHOUT this, nested validation does NOT run
```

### Test Result
```json
// Missing publisher.name
{
  "success": false,
  "data": null,
  "message": "publisher.name should not be empty, publisher.name must be a string"
}
// NestJS automatically prefixes with the path: "publisher.name"
```

---

## Part 3 — Custom Validators

When built-in validators aren't enough, write your own.

**Example:** `availableCopies` must never exceed `totalCopies`.

### Step 1 — Create the custom validator

```typescript
// src/validators/available-copies.validator.ts
import { registerDecorator, ValidationOptions, ValidationArguments } from 'class-validator';

export function IsNotMoreThan(property: string, validationOptions?: ValidationOptions) {
  return function (object: object, propertyName: string) {
    registerDecorator({
      name: 'isNotMoreThan',
      target: object.constructor,
      propertyName: propertyName,
      constraints: [property],
      options: validationOptions,
      validator: {
        validate(value: number, args: ValidationArguments) {
          const [relatedPropertyName] = args.constraints;
          const relatedValue = (args.object as any)[relatedPropertyName];
          return value <= relatedValue;  // the actual validation logic
        },
        defaultMessage(args: ValidationArguments) {
          const [relatedPropertyName] = args.constraints;
          return `${args.property} must not be greater than ${relatedPropertyName}`;
        },
      },
    });
  };
}
```

### Step 2 — Use it in the DTO

```typescript
@IsInt()
@Min(0)
@IsNotMoreThan('totalCopies', { message: 'availableCopies cannot exceed totalCopies' })
availableCopies: number;
```

### Test Result
```json
// availableCopies: 5, totalCopies: 2
{
  "success": false,
  "data": null,
  "message": "availableCopies cannot exceed totalCopies"
}
```

---

## Complete `CreateBookDto` (Final Version)

```typescript
import { IsString, IsNotEmpty, IsInt, Min, IsOptional, ValidateNested } from 'class-validator';
import { Type } from 'class-transformer';
import { PublisherDto } from './publisher.dto';
import { IsNotMoreThan } from '../../validators/available-copies.validator';

export class CreateBookDto {
  @IsString()
  @IsNotEmpty()
  title: string;

  @IsString()
  @IsNotEmpty()
  author: string;

  @IsString()
  @IsOptional()
  genre?: string;

  @IsInt()
  @Min(1)
  totalCopies: number;

  @IsInt()
  @Min(0)
  @IsNotMoreThan('totalCopies', { message: 'availableCopies cannot exceed totalCopies' })
  availableCopies: number;

  @IsOptional()
  @ValidateNested()
  @Type(() => PublisherDto)
  publisher?: PublisherDto;
}
```

---

## Common `class-validator` Decorators — Full Reference

| Decorator | Rule |
|---|---|
| `@IsString()` | Must be a string |
| `@IsNotEmpty()` | Cannot be empty `""` |
| `@IsInt()` | Must be a whole number |
| `@IsNumber()` | Must be any number |
| `@Min(n)` | Must be at least n |
| `@Max(n)` | Must be at most n |
| `@IsOptional()` | Skip validation if field is missing |
| `@IsEmail()` | Must be valid email format |
| `@IsBoolean()` | Must be boolean |
| `@IsArray()` | Must be an array |
| `@IsEnum(MyEnum)` | Must be a value from an enum |
| `@MinLength(n)` | String minimum length |
| `@MaxLength(n)` | String maximum length |
| `@ValidateNested()` | Validate a nested object |
| `@Type(() => Class)` | Tell class-transformer what type to use |

---

## File Structure After Step 11

```
src/
  books/
    dto/
      create-book.dto.ts    ← updated with nested DTO + custom validator
      update-book.dto.ts
      publisher.dto.ts      ← new nested DTO
  validators/
    available-copies.validator.ts  ← new custom validator
  main.ts                   ← transform: true added
```

---

## DotAI vs Library Tracker

| | DotAI | Library Tracker |
|---|---|---|
| DTOs location | `submodules/dtos-submodule/` | `src/books/dto/` |
| Validation | Manual in services | `class-validator` decorators |
| Nested DTOs | Used in complex payloads | `PublisherDto` inside `CreateBookDto` |
| Custom validators | Not used — manual checks | `IsNotMoreThan` custom decorator |
| Transform | Not enabled | `transform: true` in ValidationPipe |

---

## Key Takeaways

> 1. **DTO ≠ Schema** — DTO validates incoming data, Schema defines DB structure
> 2. `transform: true` — auto-converts types instead of rejecting (e.g. `"3"` → `3`)
> 3. `@ValidateNested()` + `@Type()` — both needed together to validate nested objects
> 4. Custom validators with `registerDecorator()` — write any rule built-ins can't express
> 5. Error messages automatically prefix nested paths — `publisher.name must be a string`

---

*Generated: 2026-03-23 | Project: library-tracker*
