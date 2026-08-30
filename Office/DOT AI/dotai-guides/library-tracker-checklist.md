# Library Tracker — Build Checklist

> A step-by-step checklist to build a NestJS + MongoDB REST API from scratch.
> Reference project: `library-tracker` — manages books, members, and borrowings.

---

## Phase 1 — Setup & Installation

- [ ] Check Node.js → `node -v` (need v20+)
- [ ] Check npm → `npm -v`
- [ ] Install NestJS CLI → `npm install -g @nestjs/cli`
- [ ] Install MongoDB → `brew tap mongodb/brew && brew install mongodb-community`
- [ ] Start MongoDB → `brew services start mongodb/brew/mongodb-community`
- [ ] Verify MongoDB running → `mongod --version`

---

## Phase 2 — Create the Project

- [ ] Create NestJS project → `nest new library-tracker --package-manager npm`
- [ ] Navigate into project → `cd library-tracker`
- [ ] Install Mongoose → `npm install @nestjs/mongoose mongoose`

---

## Phase 3 — Connect MongoDB

- [ ] Open `src/app.module.ts`
- [ ] Import `MongooseModule` from `@nestjs/mongoose`
- [ ] Add to `imports[]`:

```typescript
MongooseModule.forRoot('mongodb://localhost:27017/library-tracker')
```

Final `app.module.ts` should look like:

```typescript
import { Module } from '@nestjs/common';
import { MongooseModule } from '@nestjs/mongoose';
import { AppController } from './app.controller';
import { AppService } from './app.service';

@Module({
  imports: [
    MongooseModule.forRoot('mongodb://localhost:27017/library-tracker'),
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

---

## Phase 4 — Generate Modules

Run these CLI commands for each feature (books, members, borrowings):

```bash
nest generate module books
nest generate controller books --no-spec
nest generate service books --no-spec

nest generate module members
nest generate controller members --no-spec
nest generate service members --no-spec

nest generate module borrowings
nest generate controller borrowings --no-spec
nest generate service borrowings --no-spec
```

> NestJS CLI automatically imports each module into `app.module.ts` for you.

---

## Phase 5 — Build Each Module

> Follow this pattern for every module. Books is shown as the example.

---

### Step 1 — Schema (`src/books/book.schema.ts`)

Defines what a document looks like in MongoDB.

```typescript
import { Prop, Schema, SchemaFactory } from '@nestjs/mongoose';
import { HydratedDocument } from 'mongoose';

export type BookDocument = HydratedDocument<Book>;

@Schema({ timestamps: true })   // auto adds createdAt, updatedAt
export class Book {
  @Prop({ required: true })
  title: string;

  @Prop({ required: true })
  author: string;

  @Prop()
  genre: string;

  @Prop({ required: true })
  totalCopies: number;

  @Prop({ required: true })
  availableCopies: number;
}

export const BookSchema = SchemaFactory.createForClass(Book);
```

**Checklist:**
- [ ] Create `src/books/book.schema.ts`
- [ ] Add `@Schema({ timestamps: true })` on the class
- [ ] Add `@Prop()` decorator on each field
- [ ] Mark required fields with `{ required: true }`
- [ ] Export `BookDocument` type
- [ ] Export `BookSchema`

---

### Step 2 — DTOs (`src/books/dto/`)

Defines the shape of data coming in from HTTP requests.

**`create-book.dto.ts`** — all fields required:
```typescript
export class CreateBookDto {
  title: string;
  author: string;
  genre: string;
  totalCopies: number;
  availableCopies: number;
}
```

**`update-book.dto.ts`** — all fields optional (use `?`):
```typescript
export class UpdateBookDto {
  title?: string;
  author?: string;
  genre?: string;
  totalCopies?: number;
  availableCopies?: number;
}
```

**Checklist:**
- [ ] Create `src/books/dto/create-book.dto.ts` — all fields required
- [ ] Create `src/books/dto/update-book.dto.ts` — all fields optional with `?`

---

### Step 3 — Service (`src/books/books.service.ts`)

All business logic and database operations live here.

```typescript
import { Injectable } from '@nestjs/common';
import { InjectModel } from '@nestjs/mongoose';
import { Model } from 'mongoose';
import { Book, BookDocument } from './book.schema';
import { CreateBookDto } from './dto/create-book.dto';
import { UpdateBookDto } from './dto/update-book.dto';

@Injectable()
export class BooksService {
  constructor(
    @InjectModel(Book.name)
    private bookModel: Model<BookDocument>,
  ) {}

  async create(createBookDto: CreateBookDto): Promise<Book> {
    const newBook = new this.bookModel(createBookDto);
    return newBook.save();
  }

  async findAll(): Promise<Book[]> {
    return this.bookModel.find().exec();
  }

  async findOne(id: string): Promise<Book | null> {
    return this.bookModel.findById(id).exec();
  }

  async update(id: string, updateBookDto: UpdateBookDto): Promise<Book | null> {
    return this.bookModel
      .findByIdAndUpdate(id, updateBookDto, { new: true })
      .exec();
  }

  async delete(id: string): Promise<Book | null> {
    return this.bookModel.findByIdAndDelete(id).exec();
  }
}
```

**Checklist:**
- [ ] Add `@Injectable()` decorator on the class
- [ ] Inject model using `@InjectModel(Book.name)` in constructor
- [ ] Write `create()` → `new this.bookModel(dto).save()`
- [ ] Write `findAll()` → `this.bookModel.find().exec()`
- [ ] Write `findOne(id)` → `this.bookModel.findById(id).exec()`
- [ ] Write `update(id, dto)` → `findByIdAndUpdate(id, dto, { new: true }).exec()`
- [ ] Write `delete(id)` → `this.bookModel.findByIdAndDelete(id).exec()`

---

### Step 4 — Controller (`src/books/books.controller.ts`)

Handles HTTP routing only. Calls service methods. No logic here.

```typescript
import {
  Controller,
  Get,
  Post,
  Patch,
  Delete,
  Param,
  Body,
} from '@nestjs/common';
import { BooksService } from './books.service';
import { CreateBookDto } from './dto/create-book.dto';
import { UpdateBookDto } from './dto/update-book.dto';

@Controller('books')
export class BooksController {
  constructor(private readonly booksService: BooksService) {}

  @Post()
  create(@Body() createBookDto: CreateBookDto) {
    return this.booksService.create(createBookDto);
  }

  @Get()
  findAll() {
    return this.booksService.findAll();
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.booksService.findOne(id);
  }

  @Patch(':id')
  update(@Param('id') id: string, @Body() updateBookDto: UpdateBookDto) {
    return this.booksService.update(id, updateBookDto);
  }

  @Delete(':id')
  delete(@Param('id') id: string) {
    return this.booksService.delete(id);
  }
}
```

**Checklist:**
- [ ] Add `@Controller('books')` on the class
- [ ] Inject service via constructor
- [ ] Add `@Post()` → calls `service.create(@Body())`
- [ ] Add `@Get()` → calls `service.findAll()`
- [ ] Add `@Get(':id')` → calls `service.findOne(@Param('id'))`
- [ ] Add `@Patch(':id')` → calls `service.update(@Param('id'), @Body())`
- [ ] Add `@Delete(':id')` → calls `service.delete(@Param('id'))`

---

### Step 5 — Module (`src/books/books.module.ts`)

Wires the schema, controller, and service together.

```typescript
import { Module } from '@nestjs/common';
import { MongooseModule } from '@nestjs/mongoose';
import { BooksController } from './books.controller';
import { BooksService } from './books.service';
import { Book, BookSchema } from './book.schema';

@Module({
  imports: [
    MongooseModule.forFeature([{ name: Book.name, schema: BookSchema }]),
  ],
  controllers: [BooksController],
  providers: [BooksService],
  exports: [BooksService],   // export so BorrowingsModule can use BooksService
})
export class BooksModule {}
```

**Checklist:**
- [ ] Import `MongooseModule.forFeature([{ name: Book.name, schema: BookSchema }])` in `imports[]`
- [ ] Add controller to `controllers[]`
- [ ] Add service to `providers[]`
- [ ] Add service to `exports[]`

---

## Phase 6 — Test

- [ ] Start app → `npm run start:dev`
- [ ] Check logs — confirm all routes are mapped, no errors
- [ ] Test `POST /books`:
  ```bash
  curl -X POST http://localhost:3000/books \
    -H "Content-Type: application/json" \
    -d '{"title":"The Pragmatic Programmer","author":"Andy Hunt","genre":"Technology","totalCopies":3,"availableCopies":3}'
  ```
- [ ] Test `GET /books` → should return array with the created book
- [ ] Test `GET /books/:id` → paste the `_id` from the created book
- [ ] Test `PATCH /books/:id` → update a field
- [ ] Test `DELETE /books/:id` → delete the book
- [ ] Open MongoDB Compass → connect to `mongodb://localhost:27017` → verify data in `library-tracker` database

---

## Module Pattern — Quick Reference

Every module follows this exact same 5-file pattern:

```
books/
  book.schema.ts          ← what the DB document looks like
  dto/
    create-book.dto.ts    ← shape of POST request body
    update-book.dto.ts    ← shape of PATCH request body (all optional)
  books.service.ts        ← DB operations (create, find, update, delete)
  books.controller.ts     ← HTTP routes (calls service methods)
  books.module.ts         ← wires everything together
```

---

## Members Module ✅

Same pattern as Books. Key difference: `email` has `unique: true` in schema.

**Schema** (`src/members/member.schema.ts`):
```typescript
import { Prop, Schema, SchemaFactory } from '@nestjs/mongoose';
import { HydratedDocument } from 'mongoose';

export type MemberDocument = HydratedDocument<Member>;

@Schema({ timestamps: true })
export class Member {
  @Prop({ required: true })
  name: string;

  @Prop({ required: true, unique: true })  // unique: true = no duplicate emails
  email: string;

  @Prop()
  phone: string;
}

export const MemberSchema = SchemaFactory.createForClass(Member);
```

**Checklist:**
- [ ] Create `src/members/member.schema.ts` — add `unique: true` on email
- [ ] Create `src/members/dto/create-member.dto.ts` — `phone` is optional (`?`)
- [ ] Create `src/members/dto/update-member.dto.ts` — all fields optional
- [ ] Write `members.service.ts` — same 5 methods as BooksService
- [ ] Write `members.controller.ts` — same 5 routes as BooksController
- [ ] Update `members.module.ts` — register schema, add `exports: [MembersService]`

**Test:**
```bash
# Create a member
curl -X POST http://localhost:3000/members \
  -H "Content-Type: application/json" \
  -d '{"name":"John Doe","email":"john@example.com","phone":"1234567890"}'

# Get all members
curl http://localhost:3000/members
```

---

## Borrowings Module ✅

Most complex module — injects `BooksService` and `MembersService` via DI.
Introduces: cross-module DI, throwing HTTP exceptions, `.populate()` for joins.

---

### Schema (`src/borrowings/borrowing.schema.ts`)

```typescript
import { Prop, Schema, SchemaFactory } from '@nestjs/mongoose';
import { HydratedDocument, Types } from 'mongoose';

export type BorrowingDocument = HydratedDocument<Borrowing>;

@Schema({ timestamps: true })
export class Borrowing {
  @Prop({ type: Types.ObjectId, ref: 'Member', required: true })
  memberId: Types.ObjectId;       // ref: 'Member' = foreign key pointing to Member collection

  @Prop({ type: Types.ObjectId, ref: 'Book', required: true })
  bookId: Types.ObjectId;         // ref: 'Book' = foreign key pointing to Book collection

  @Prop({ default: false })
  returned: boolean;              // default: false = auto set on creation

  @Prop()
  returnedAt: Date;
}

export const BorrowingSchema = SchemaFactory.createForClass(Borrowing);
```

**Checklist:**
- [ ] Create `src/borrowings/borrowing.schema.ts`
- [ ] Use `Types.ObjectId` + `ref:` to create references to other collections
- [ ] Add `returned: boolean` with `default: false`
- [ ] Add `returnedAt: Date` (set when book is returned)

---

### DTO (`src/borrowings/dto/create-borrowing.dto.ts`)

```typescript
export class CreateBorrowingDto {
  memberId: string;
  bookId: string;
}
```

**Checklist:**
- [ ] Create `src/borrowings/dto/create-borrowing.dto.ts` — just `memberId` and `bookId`

---

### Service (`src/borrowings/borrowings.service.ts`)

```typescript
import { Injectable, BadRequestException, NotFoundException } from '@nestjs/common';
import { InjectModel } from '@nestjs/mongoose';
import { Model } from 'mongoose';
import { Borrowing, BorrowingDocument } from './borrowing.schema';
import { CreateBorrowingDto } from './dto/create-borrowing.dto';
import { BooksService } from '../books/books.service';
import { MembersService } from '../members/members.service';

@Injectable()
export class BorrowingsService {
  constructor(
    @InjectModel(Borrowing.name)
    private borrowingModel: Model<BorrowingDocument>,

    // cross-module DI — injecting services from other modules
    private booksService: BooksService,
    private membersService: MembersService,
  ) {}

  async borrow(createBorrowingDto: CreateBorrowingDto): Promise<Borrowing> {
    const { memberId, bookId } = createBorrowingDto;

    // 1. Check member exists
    const member = await this.membersService.findOne(memberId);
    if (!member) throw new NotFoundException(`Member with ID ${memberId} not found`);

    // 2. Check book exists
    const book = await this.booksService.findOne(bookId);
    if (!book) throw new NotFoundException(`Book with ID ${bookId} not found`);

    // 3. Check book has available copies
    if (book.availableCopies <= 0) throw new BadRequestException(`No available copies`);

    // 4. Decrease available copies by 1
    await this.booksService.update(bookId, { availableCopies: book.availableCopies - 1 });

    // 5. Create borrowing record
    const newBorrowing = new this.borrowingModel({ memberId, bookId });
    return newBorrowing.save();
  }

  async returnBook(borrowingId: string): Promise<Borrowing | null> {
    // 1. Find the borrowing record
    const borrowing = await this.borrowingModel.findById(borrowingId);
    if (!borrowing) throw new NotFoundException(`Borrowing record not found`);

    // 2. Check it hasn't already been returned
    if (borrowing.returned) throw new BadRequestException(`Book already returned`);

    // 3. Increase available copies by 1
    const book = await this.booksService.findOne(borrowing.bookId.toString());
    if (book) {
      await this.booksService.update(borrowing.bookId.toString(), {
        availableCopies: book.availableCopies + 1,
      });
    }

    // 4. Mark as returned
    return this.borrowingModel
      .findByIdAndUpdate(borrowingId, { returned: true, returnedAt: new Date() }, { new: true })
      .exec();
  }

  async findAll(): Promise<Borrowing[]> {
    return this.borrowingModel
      .find()
      .populate('memberId', 'name email')   // replaces ID with member name + email
      .populate('bookId', 'title author')   // replaces ID with book title + author
      .exec();
  }
}
```

**Checklist:**
- [ ] Inject `BooksService` and `MembersService` in constructor (no `@InjectModel` needed — just plain DI)
- [ ] `borrow()`: check member exists → check book exists → check availableCopies > 0 → decrement → save
- [ ] `returnBook()`: find borrowing → check not already returned → increment copies → mark returned
- [ ] `findAll()`: use `.populate()` to replace IDs with actual member/book data
- [ ] Use `NotFoundException` (404) and `BadRequestException` (400) for error handling

---

### Controller (`src/borrowings/borrowings.controller.ts`)

```typescript
import { Controller, Get, Post, Patch, Param, Body } from '@nestjs/common';
import { BorrowingsService } from './borrowings.service';
import { CreateBorrowingDto } from './dto/create-borrowing.dto';

@Controller('borrowings')
export class BorrowingsController {
  constructor(private readonly borrowingsService: BorrowingsService) {}

  @Post()                          // POST /borrowings — borrow a book
  borrow(@Body() createBorrowingDto: CreateBorrowingDto) {
    return this.borrowingsService.borrow(createBorrowingDto);
  }

  @Patch(':id/return')             // PATCH /borrowings/:id/return — return a book
  returnBook(@Param('id') id: string) {
    return this.borrowingsService.returnBook(id);
  }

  @Get()                           // GET /borrowings — list all with populated data
  findAll() {
    return this.borrowingsService.findAll();
  }
}
```

**Checklist:**
- [ ] `@Post()` → calls `service.borrow()`
- [ ] `@Patch(':id/return')` → calls `service.returnBook(id)`
- [ ] `@Get()` → calls `service.findAll()`

---

### Module (`src/borrowings/borrowings.module.ts`)

```typescript
import { Module } from '@nestjs/common';
import { MongooseModule } from '@nestjs/mongoose';
import { BorrowingsController } from './borrowings.controller';
import { BorrowingsService } from './borrowings.service';
import { Borrowing, BorrowingSchema } from './borrowing.schema';
import { BooksModule } from '../books/books.module';
import { MembersModule } from '../members/members.module';

@Module({
  imports: [
    MongooseModule.forFeature([{ name: Borrowing.name, schema: BorrowingSchema }]),
    BooksModule,    // import BooksModule to get access to BooksService
    MembersModule,  // import MembersModule to get access to MembersService
  ],
  controllers: [BorrowingsController],
  providers: [BorrowingsService],
})
export class BorrowingsModule {}
```

**Checklist:**
- [ ] Import `BooksModule` and `MembersModule` in `imports[]`
- [ ] This works because both modules have `exports: [BooksService]` and `exports: [MembersService]`

---

### Test the Full Flow

```bash
# 1. Borrow a book (use real IDs from your DB)
curl -X POST http://localhost:3000/borrowings \
  -H "Content-Type: application/json" \
  -d '{"memberId":"<memberId>","bookId":"<bookId>"}'
# Expected: returned: false, availableCopies decreases by 1

# 2. Check available copies decreased
curl http://localhost:3000/books/<bookId>

# 3. Return the book
curl -X PATCH http://localhost:3000/borrowings/<borrowingId>/return
# Expected: returned: true, returnedAt set, availableCopies increases by 1

# 4. Get all borrowings — member and book data populated
curl http://localhost:3000/borrowings
# Expected: memberId shows { name, email }, bookId shows { title, author }
```

---

## New Concepts Introduced in Borrowings Module

| Concept | Code | What it does |
|---|---|---|
| Cross-module DI | `imports: [BooksModule, MembersModule]` | Use services from other modules |
| References | `@Prop({ type: Types.ObjectId, ref: 'Book' })` | Foreign key to another collection |
| Populate | `.populate('bookId', 'title author')` | Fetch related document (like a JOIN) |
| HTTP Exceptions | `throw new NotFoundException(...)` | NestJS auto-sends 404/400 response |
| Default values | `@Prop({ default: false })` | Auto-set field value on creation |

---

## Key Concepts Used

| Concept | Where |
|---|---|
| `@Module()` | `books.module.ts` |
| `@Controller()` | `books.controller.ts` |
| `@Injectable()` | `books.service.ts` |
| Dependency Injection | constructor in service and controller |
| `@InjectModel()` | injecting Mongoose model into service |
| `@Schema()`, `@Prop()` | defining MongoDB schema |
| DTOs | shaping request data |
| `MongooseModule.forFeature()` | registering schema in module |
| `MongooseModule.forRoot()` | connecting to MongoDB in root module |

---

*Generated: 2026-03-22 | Project: library-tracker*
