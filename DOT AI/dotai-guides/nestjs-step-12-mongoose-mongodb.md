# Step 12 — Mongoose / MongoDB Integration (Deep Dive)

> Stage 3: Data & Config | Project: library-tracker

---

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

## DotAI vs Library Tracker

| | Library Tracker | DotAI |
|---|---|---|
| Schema location | `src/books/book.schema.ts` | `submodules/entities-submodule/` |
| Model injection | `@InjectModel(Book.name)` | Same via `createTenantModelProvider` |
| Relationships | `Types.ObjectId` + `.populate()` | Same, across tenant DBs |
| Hooks | Not used | Post-save hooks → `*_history` collections |
| Aggregation | Not used | Used for reports and analytics |
| Indexes | Not defined | Defined on every queryable field |

---

## Key Takeaways

> 1. `@Prop()` options are MongoDB-level — `required`, `unique`, `default`, `enum`, `index`
> 2. `{ new: true }` — always include when you want the updated document back
> 3. `.populate()` — Mongoose makes a second query to replace the ObjectId with the real document
> 4. Indexes — add on any field you query/sort by frequently
> 5. Hooks (`pre`/`post`) — run logic automatically on DB operations (audit trails in DotAI)
> 6. Aggregation — use `.aggregate()` for joins, grouping, computed fields

---

*Generated: 2026-03-23 | Project: library-tracker*
