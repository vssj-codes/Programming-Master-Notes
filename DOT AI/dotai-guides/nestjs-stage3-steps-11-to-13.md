# Stage 3 — Data & Config (Steps 11–13)

> Project: library-tracker

---

# Step 11 — DTOs & Validation (Deep Dive)

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

## DotAI vs Library Tracker (Step 11)

| | DotAI | Library Tracker |
|---|---|---|
| DTOs location | `submodules/dtos-submodule/` | `src/books/dto/` |
| Validation | Manual in services | `class-validator` decorators |
| Nested DTOs | Used in complex payloads | `PublisherDto` inside `CreateBookDto` |
| Custom validators | Not used — manual checks | `IsNotMoreThan` custom decorator |
| Transform | Not enabled | `transform: true` in ValidationPipe |

## Key Takeaways — Step 11

> 1. **DTO ≠ Schema** — DTO validates incoming data, Schema defines DB structure
> 2. `transform: true` — auto-converts types instead of rejecting (e.g. `"3"` → `3`)
> 3. `@ValidateNested()` + `@Type()` — both needed together to validate nested objects
> 4. Custom validators with `registerDecorator()` — write any rule built-ins can't express
> 5. Error messages automatically prefix nested paths — `publisher.name must be a string`

---

---

# Step 12 — Mongoose / MongoDB Integration (Deep Dive)

## Part 1 — `@Schema()` and `@Prop()` Options

```typescript
@Schema({
  timestamps: true,      // auto-adds createdAt + updatedAt
  collection: 'books',   // override MongoDB collection name
  versionKey: false,     // remove __v field
  toJSON: { virtuals: true },  // include virtual fields in responses
})
export class Book {

  @Prop({ required: true })
  title: string;

  @Prop({ unique: true })          // MongoDB enforces uniqueness at DB level
  isbn: string;

  @Prop({ default: 'Unknown' })    // value if field not provided
  genre: string;

  @Prop({ enum: ['Fiction', 'Non-Fiction', 'Science'] })
  category: string;

  @Prop({ min: 1, max: 1000 })    // number bounds at DB level
  totalCopies: number;

  @Prop({ index: true })           // DB index — faster queries on this field
  author: string;

  @Prop({ select: false })         // never returned in queries by default
  internalNotes: string;
}
```

**Key distinction:** `@Prop()` options = MongoDB-level constraints. `class-validator` decorators on DTOs = application-level constraints. Both exist independently.

---

## Part 2 — Model Methods Reference

```typescript
// --- BASIC QUERIES ---
await this.bookModel.find();
await this.bookModel.find({ genre: 'Fiction' });
await this.bookModel.findById(id);
await this.bookModel.findOne({ title: 'Dune' });

// --- SELECT ---
await this.bookModel.find().select('title author');   // only these fields
await this.bookModel.find().select('-genre');          // all EXCEPT genre

// --- SORT, SKIP, LIMIT (pagination) ---
await this.bookModel.find()
  .sort({ createdAt: -1 })
  .skip(20)
  .limit(10);

// --- CREATE ---
const newBook = new this.bookModel(dto);
await newBook.save();
// OR:
await this.bookModel.create(dto);

// --- UPDATE ---
await this.bookModel.findByIdAndUpdate(
  id,
  { availableCopies: 5 },
  { new: true }   // return updated doc (not old doc)
);
await this.bookModel.updateMany({ genre: 'Fiction' }, { $set: { featured: true } });

// --- DELETE ---
await this.bookModel.findByIdAndDelete(id);
await this.bookModel.deleteMany({ genre: 'Old' });

// --- COUNT ---
await this.bookModel.countDocuments({ genre: 'Fiction' });
```

---

## Part 3 — `{ new: true }` — Why It Matters

```typescript
// WITHOUT { new: true } → returns document BEFORE the update
await this.bookModel.findByIdAndUpdate(id, { returned: true });

// WITH { new: true } → returns document AFTER the update
await this.bookModel.findByIdAndUpdate(id, { returned: true }, { new: true });
```

Always use `{ new: true }` when you want to return the updated result to the client.

---

## Part 4 — Relationships & `.populate()`

```typescript
// Schema definition (borrowing.schema.ts):
@Prop({ type: Types.ObjectId, ref: 'Member', required: true })
memberId: Types.ObjectId;

// Querying with population:
await this.borrowingModel
  .find()
  .populate('memberId', 'name email')   // replace ObjectId with member document
  .populate('bookId', 'title author')   // replace ObjectId with book document
  .exec();
```

**What happens under the hood:**
```
Stored in DB:
{ memberId: ObjectId("abc123"), bookId: ObjectId("def456"), returned: false }

After .populate('memberId', 'name email'):
{ memberId: { _id: "abc123", name: "Alice", email: "alice@x.com" }, ... }
```

Mongoose makes a **second DB query** automatically. The second argument is a field projection — only return those fields.

---

## Part 5 — Indexes

Without an index: MongoDB scans every document (O(n)).
With an index: O(log n).

```typescript
// Single-field index on the property
@Prop({ index: true })
author: string;

@Prop({ unique: true })   // unique: true also creates an index automatically
email: string;

// Compound index on the schema (after SchemaFactory call)
BookSchema.index({ author: 1, genre: 1 });  // fast for queries filtering by both
```

**When to add indexes:**
- Fields you query by frequently
- Fields used in `sort()` calls
- Fields with `unique: true` (auto-indexed)

---

## Part 6 — Mongoose Middleware Hooks

Hooks run automatically before or after Mongoose operations.

```typescript
// In the schema file, AFTER SchemaFactory.createForClass():
export const BookSchema = SchemaFactory.createForClass(Book);

// Pre-save hook
BookSchema.pre('save', function (next) {
  console.log('About to save:', this.title);
  next();
});

// Post-save hook
BookSchema.post('save', function (doc) {
  console.log('Saved with id:', doc._id);
  // DotAI pattern: write audit record to book_history collection here
});

// Pre-findOneAndUpdate hook
BookSchema.pre('findOneAndUpdate', function (next) {
  this.set({ updatedAt: new Date() });
  next();
});
```

**DotAI uses this pattern** to write to `vehicle_history`, `asset_history` etc. — every mutation is recorded automatically via a post-save hook, not manually in every service.

---

## Part 7 — Virtuals

A virtual is a computed field — exists on the JavaScript object, never stored in MongoDB.

```typescript
BookSchema.virtual('isBorrowable').get(function () {
  return this.availableCopies > 0;
});
```

To include virtuals in JSON responses, add to `@Schema()`:
```typescript
@Schema({ toJSON: { virtuals: true } })
```

---

## Part 8 — Aggregation Pipeline

Use `.aggregate()` for joins, grouping, computed fields. Use `.find()` for everything else.

```typescript
const results = await this.borrowingModel.aggregate([
  // Stage 1: JOIN borrowings → books
  {
    $lookup: {
      from: 'books',         // MongoDB collection name
      localField: 'bookId',
      foreignField: '_id',
      as: 'book',            // output field (always an array)
    },
  },

  // Stage 2: flatten the array from $lookup
  { $unwind: '$book' },

  // Stage 3: JOIN borrowings → members
  {
    $lookup: {
      from: 'members',
      localField: 'memberId',
      foreignField: '_id',
      as: 'member',
    },
  },
  { $unwind: '$member' },

  // Stage 4: shape the output (like SELECT)
  {
    $project: {
      _id: 1,
      returned: 1,
      'book.title': 1,
      'book.author': 1,
      'member.name': 1,
      'member.email': 1,
      status: {
        $cond: { if: '$returned', then: 'Returned', else: 'Active' }
      }
    },
  },

  // Stage 5: filter (can filter on computed fields — unlike .find())
  { $match: { returned: false } },

  // Stage 6: sort
  { $sort: { createdAt: -1 } },
]);
```

---

## DotAI vs Library Tracker (Step 12)

| | Library Tracker | DotAI |
|---|---|---|
| Schema location | `src/books/book.schema.ts` | `submodules/entities-submodule/` |
| Model injection | `@InjectModel(Book.name)` | Same via `createTenantModelProvider` |
| Relationships | `Types.ObjectId` + `.populate()` | Same, across tenant DBs |
| Hooks | Not used | Post-save hooks → `*_history` collections |
| Aggregation | Not used | Used for reports and analytics |
| Indexes | Not defined | Defined on every queryable field |

## Key Takeaways — Step 12

> 1. `@Prop()` options are MongoDB-level — `required`, `unique`, `default`, `enum`, `index`
> 2. `{ new: true }` — always include when you want the updated document back
> 3. `.populate()` — Mongoose makes a second query to replace the ObjectId with the real document
> 4. Indexes — add on any field you query/sort by frequently
> 5. Hooks (`pre`/`post`) — run logic automatically on DB operations (audit trails in DotAI)
> 6. Aggregation — use `.aggregate()` for joins, grouping, computed fields

---

---

# Step 13 — Environment Config

*To be added.*

---

*Generated: 2026-03-23 | Project: library-tracker*
