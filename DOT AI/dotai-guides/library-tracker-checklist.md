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

## Still To Build

- [ ] Members module (same pattern as Books)
- [ ] Borrowings module (uses BooksService + MembersService via DI)

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
