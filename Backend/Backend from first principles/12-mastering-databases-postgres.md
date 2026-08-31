# Table-of-Contents

<!-- toc -->

- [Mastering Databases with PostgreSQL](#mastering-databases-with-postgresql)
  * [Why Databases Exist](#why-databases-exist)
  * [What Is a Database?](#what-is-a-database)
  * [Disk vs RAM Storage](#disk-vs-ram-storage)
  * [DBMS (Database Management System)](#dbms-database-management-system)
    + [Why Not Just Text Files?](#why-not-just-text-files)
  * [Relational vs Non-Relational](#relational-vs-non-relational)
    + [Relational (SQL)](#relational-sql)
    + [Non-Relational (NoSQL)](#non-relational-nosql)
  * [Why PostgreSQL](#why-postgresql)
  * [PostgreSQL Data Types](#postgresql-data-types)
    + [Numeric](#numeric)
    + [String](#string)
    + [Other Types](#other-types)
  * [Migrations](#migrations)
    + [Structure](#structure)
    + [Up and Down Migrations](#up-and-down-migrations)
    + [Why Migrations?](#why-migrations)
  * [Database Modeling — Project Management Platform](#database-modeling--project-management-platform)
    + [Enum Types](#enum-types)
    + [Naming Conventions](#naming-conventions)
    + [Constraints](#constraints)
    + [Default Fields Every Table Should Have](#default-fields-every-table-should-have)
  * [Relationships](#relationships)
    + [One-to-One](#one-to-one)
    + [One-to-Many](#one-to-many)
    + [Many-to-Many](#many-to-many)
  * [Referential Integrity (ON DELETE)](#referential-integrity-on-delete)
  * [Seeding](#seeding)
  * [Writing Queries for APIs](#writing-queries-for-apis)
    + [Parameterized Queries (Preventing SQL Injection)](#parameterized-queries-preventing-sql-injection)
    + [Get All Users (with JOIN)](#get-all-users-with-join)
    + [Get Single User](#get-single-user)
    + [Create User](#create-user)
    + [Update User Profile](#update-user-profile)
    + [Dynamic Filtering, Sorting, Pagination](#dynamic-filtering-sorting-pagination)
  * [Indexes](#indexes)
    + [Without Index (Sequential Scan)](#without-index-sequential-scan)
    + [With Index](#with-index)
    + [When to Create Indexes](#when-to-create-indexes)
    + [Index Syntax](#index-syntax)
    + [What's Automatically Indexed](#whats-automatically-indexed)
    + [Index Trade-off](#index-trade-off)
  * [Triggers](#triggers)
    + [Example: Auto-update `updated_at`](#example-auto-update-updated_at)
  * [Summary](#summary)

<!-- tocstop -->

---

# Mastering Databases with PostgreSQL

**Source:** Sriniously — Backend from First Principles (Video 12)
**Link:** [Watch](https://www.youtube.com/watch?v=F7Vwp2Xo5Do)

---

## Why Databases Exist

**Persistence** = storing data so it survives after the program stops. You add items to a to-do app, close it, reopen it — everything is still there. Without persistence, you'd lose all progress every time.

---

## What Is a Database?

Surprisingly broad — any **structured storage** counts:
- Phone contact list
- Browser localStorage / sessionStorage / cookies
- A simple text file with notes

**Pattern:** A system that offers ways to **create, read, update, and delete** (CRUD) data from a persistent store.

In backend context, "database" specifically means **disk-based databases**.

---

## Disk vs RAM Storage

| | RAM (Primary Memory) | Disk (Secondary Memory) |
|---|---|---|
| **Speed** | Very fast | Slower |
| **Capacity** | 8–128 GB typical | 512 GB – 2 TB typical |
| **Cost** | Expensive | Cheap |
| **Used by** | Caching (Redis, in-memory) | Databases (PostgreSQL, MongoDB) |

**Trade-off:** Databases need large capacity at affordable cost → disk storage. Speed trade-off is acceptable. Caching (Redis) uses RAM for frequently accessed data where speed matters most.

---

## DBMS (Database Management System)

Software whose responsibilities are:

1. **Data organization** — efficiently store data for fast CRUD
2. **Access** — provide methods for create, read, update, delete
3. **Integrity** — ensure data accuracy and validity (e.g. a price field only accepts numbers)
4. **Security** — protect data from unauthorized access (users, roles, permissions)

### Why Not Just Text Files?

| Problem | Why text files fail |
|---------|-------------------|
| **Parsing** | Must write custom code to split lines, compare fields — slow and error-prone |
| **No structure** | Can't enforce data types, constraints, or consistency |
| **Concurrency** | Two users updating simultaneously → race conditions, data corruption |

---

## Relational vs Non-Relational

### Relational (SQL)

- Data organized in **tables, rows, columns**
- **Predefined strict schema** — columns and data types defined before inserting data
- Relationships via **foreign keys**
- Query language: **SQL**
- Examples: **PostgreSQL**, MySQL, SQLite, SQL Server

**Best for:** Data integrity, complex queries, relationships — CRM systems, financial data, any structured data.

### Non-Relational (NoSQL)

- Flexible schema — each document can have different structure
- No predefined schema required
- Tables = **collections**, rows = **documents**
- Examples: **MongoDB**, DynamoDB

**Best for:** Prototyping, unstructured content — CMS systems, logging, content with varying formats.

> **Caveat:** Flexible schema means data integrity must be enforced in application code — more complexity, more error-prone.

---

## Why PostgreSQL

1. **Open source and free** — host it yourself, inspect the source
2. **SQL standard compliant** — easy migration to MySQL/others later
3. **Extensible** — ~1400 pages of docs, rich extension ecosystem
4. **Reliable and scalable** — battle-tested in production
5. **Native JSON support** — `jsonb` type with indexing and querying eliminates most reasons to use MongoDB

> PostgreSQL is the #1 choice for most startups and many large companies. Don't overthink MySQL vs PostgreSQL until you're serving millions of users with specific bottlenecks.

---

## PostgreSQL Data Types

### Numeric

| Type | Use case |
|------|----------|
| `serial` / `bigserial` | Auto-incrementing integer IDs |
| `smallint`, `integer`, `bigint` | Integers of increasing capacity |
| `decimal(10,2)` / `numeric` | **Exact** precision — use for prices, money, anything accuracy-critical |
| `real`, `double precision`, `float` | **Approximate** — faster computation, use for sizes, scientific data where tiny discrepancies are OK |

> **Rule:** If accuracy matters (price, money) → `decimal`. If speed matters and tiny errors are OK → `float`.

### String

| Type | Behavior |
|------|----------|
| `char(n)` | Fixed length — pads with spaces. **Avoid** unless length is always the same (e.g. 2-letter country codes) |
| `varchar(n)` | Variable length up to `n`. The `255` is a MySQL convention — **meaningless in PostgreSQL** |
| `text` | Variable length, no limit. **Recommended by PostgreSQL docs** |

> **Always use `text`** in PostgreSQL. No performance difference vs `varchar`. Avoids unnecessary migrations when you need to increase length. Enforce length in application code.

### Other Types

| Type | Purpose |
|------|---------|
| `boolean` | `true` / `false` |
| `date` | Date only |
| `time` | Time only (HH:MM:SS) |
| `timestamp` | Date + time |
| `timestamptz` | Date + time + timezone |
| `interval` | Durations (`10 days`, `1 week`) |
| `uuid` | Universal unique identifier — popular for primary keys |
| `json` | Stored as plain text |
| `jsonb` | Binary JSON — **prefer this** for better performance and query capabilities |
| `integer[]`, `text[]` | Arrays of any type |

---

## Migrations

Version-controlled database changes applied sequentially via a CLI tool (dbmate, golang-migrate, etc.).

### Structure

```
db/
  migrations/
    20240101120000_create_users_table.sql
    20240102130000_seed_data.sql
    20240103140000_add_indexes.sql
```

### Up and Down Migrations

```sql
-- migrate:up
CREATE TABLE users (...);

-- migrate:down
DROP TABLE users;
```

- **Up** — apply changes (create tables, add columns, create indexes)
- **Down** — revert changes (drop tables, remove columns) for rollbacks

### Why Migrations?

- **Track changes** over time — committed to git with your code
- **Rollback** — revert to previous database state if something breaks
- **Schema table** — the tool maintains a `schema_migrations` table tracking the current version

---

## Database Modeling — Project Management Platform

### Enum Types

Define allowed values at the database level for data integrity AND documentation:

```sql
CREATE TYPE project_status AS ENUM ('active', 'completed', 'archived');
CREATE TYPE task_status AS ENUM ('pending', 'in_progress', 'completed', 'cancelled');
CREATE TYPE member_role AS ENUM ('owner', 'admin', 'member');
```

**Why enums over plain text:**
1. **Data integrity** — database rejects invalid values (no application code needed)
2. **Documentation** — new team members see all allowed values at a glance in the migration file

### Naming Conventions

- Table names: **plural, lowercase, snake_case** — `users`, `user_profiles`, `project_members`
- Column names: **lowercase, snake_case** — `full_name`, `created_at`, `password_hash`
- PostgreSQL is **case-insensitive by default** — camelCase requires double quotes everywhere (ugly in application code)

### Constraints

| Constraint | What it does |
|-----------|-------------|
| `PRIMARY KEY` | Unique + NOT NULL. Identifies each row. Auto-indexed by database |
| `NOT NULL` | Field cannot be null. **Use on 70%+ of fields** |
| `UNIQUE` | No duplicate values in the column (e.g. email) |
| `CHECK` | Custom condition — e.g. `CHECK (priority >= 1 AND priority <= 5)` |
| `REFERENCES` (Foreign Key) | Value must exist in the referenced table |
| `DEFAULT` | Auto-set value if not provided — e.g. `DEFAULT now()`, `DEFAULT 'active'` |

### Default Fields Every Table Should Have

```sql
id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
```

---

## Relationships

### One-to-One

Example: `users` ↔ `user_profiles`

```sql
CREATE TABLE user_profiles (
  user_id UUID PRIMARY KEY REFERENCES users(id),  -- FK is also the PK
  avatar_url TEXT,
  bio TEXT,
  phone TEXT
);
```

**Why separate tables?** Profile info changes frequently and may grow with new fields — isolating it avoids constant migrations to the main `users` table.

### One-to-Many

Example: `projects` → `tasks` (one project has many tasks)

```sql
CREATE TABLE tasks (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
  ...
);
```

The foreign key (`project_id`) stays in the "many" table, referencing the "one" table's primary key.

### Many-to-Many

Example: `projects` ↔ `users` (a user can be in many projects, a project can have many users)

Implemented via a **linking table** with a **composite primary key**:

```sql
CREATE TABLE project_members (
  project_id UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  role member_role NOT NULL DEFAULT 'member',
  PRIMARY KEY (project_id, user_id)  -- composite PK = unique + not null
);
```

The composite primary key ensures a user can only appear once per project.

---

## Referential Integrity (ON DELETE)

Protects data across related tables when a referenced row is deleted:

| Constraint | Behavior |
|-----------|----------|
| `ON DELETE RESTRICT` | **Block** the delete if referenced rows exist |
| `ON DELETE CASCADE` | **Delete** all referencing rows too |
| `ON DELETE SET NULL` | Set the foreign key to `NULL` (field must allow null) |
| `ON DELETE SET DEFAULT` | Set the foreign key to its default value |

**Example:** `ON DELETE RESTRICT` on `projects.owner_id` → can't delete a user who owns projects. Must delete their projects first.

---

## Seeding

Inserting test data into the database for development/testing. Create a separate migration file:

```sql
-- migrate:up
WITH inserted_users AS (
  INSERT INTO users (email, full_name, password_hash)
  VALUES ('alice@test.com', 'Alice Brown', 'hash1'),
         ('john@test.com', 'John Doe', 'hash2')
  RETURNING id, email
)
INSERT INTO user_profiles (user_id, bio)
SELECT id, 'Some bio' FROM inserted_users;
```

Uses CTEs (Common Table Expressions) for readable multi-table inserts.

---

## Writing Queries for APIs

### Parameterized Queries (Preventing SQL Injection)

**Never** concatenate user input into SQL strings. Use parameter slots:

```sql
SELECT * FROM users WHERE id = :user_id
```

Whatever is passed into `:user_id` is treated as a **string** — not executable SQL. This prevents SQL injection attacks like `; DELETE FROM users;`.

In application code, your database driver/ORM handles parameter binding.

### Get All Users (with JOIN)

```sql
SELECT u.*, to_jsonb(up.*) AS profile
FROM users u
LEFT JOIN user_profiles up ON u.id = up.user_id
ORDER BY u.created_at DESC;
```

- **LEFT JOIN** — returns user even if no profile exists (vs INNER JOIN which requires both)
- **to_jsonb()** — converts the profile row into an embedded JSON object
- **ORDER BY DESC** — latest users first (always apply a default sort)

### Get Single User

Same query + `WHERE u.id = :user_id`

### Create User

```sql
INSERT INTO users (email, full_name, password_hash)
VALUES (:email, :name, :password)
RETURNING *;
```

`RETURNING *` gives back the created row (with generated `id`, `created_at`).

### Update User Profile

```sql
UPDATE user_profiles
SET bio = :bio, phone = :phone
WHERE user_id = :user_id
RETURNING *;
```

Only update the fields the user actually passed (constructed dynamically in application code).

### Dynamic Filtering, Sorting, Pagination

```sql
SELECT u.*, to_jsonb(up.*) AS profile
FROM users u
LEFT JOIN user_profiles up ON u.id = up.user_id
WHERE u.full_name ILIKE :letter || '%'    -- filter: first letter (case-insensitive)
ORDER BY :sort_by :sort_order              -- dynamic sort (constructed in app code)
LIMIT :limit OFFSET :page;                 -- pagination
```

**Defaults set in application code (not DB):**

| Param | Default |
|-------|---------|
| `page` | `0` (offset 0 = first page) |
| `limit` | `10` or `20` |
| `sort_by` | `created_at` |
| `sort_order` | `DESC` |

> User-facing page numbers start at 1, but SQL `OFFSET` starts at 0. Convert in application code.

---

## Indexes

A **lookup table** that maps field values → row locations on disk, enabling direct access instead of scanning every row.

### Without Index (Sequential Scan)

Database checks every row one by one: *"Is this the ID? No. This one? No. This one? Yes."*
With 1 million rows → very slow.

### With Index

Database looks up the ID in the index → gets the disk location → jumps directly to the row.
Like a book's table of contents — jump to page 54 instead of flipping through 50 pages.

### When to Create Indexes

Create an index when a field is used in:

1. **JOIN conditions** — `ON u.id = t.assigned_to`
2. **WHERE clauses** — `WHERE status = 'pending'`
3. **ORDER BY** — `ORDER BY created_at DESC`

AND the query is **frequently executed**.

### Index Syntax

```sql
CREATE INDEX idx_users_email ON users (email);
CREATE INDEX idx_users_created_at ON users (created_at DESC);
CREATE INDEX idx_tasks_project_id ON tasks (project_id);
CREATE INDEX idx_tasks_status ON tasks (status);
```

### What's Automatically Indexed

**Primary keys** are automatically indexed — don't create manual indexes for them.

**Foreign keys are NOT automatically indexed** — create indexes for foreign keys involved in joins.

### Index Trade-off

Indexes speed up reads but add overhead to writes — every INSERT/UPDATE must also update the index. Evaluate:
- How frequently is the query called?
- Is the performance gain worth the write overhead?
- You can always drop an index later if it's not needed.

---

## Triggers

Automate database operations when certain events occur.

### Example: Auto-update `updated_at`

```sql
CREATE OR REPLACE FUNCTION update_timestamp()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = now();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER set_updated_at
  BEFORE UPDATE ON users
  FOR EACH ROW
  EXECUTE FUNCTION update_timestamp();
```

Create the function once, then create a trigger for each table. Now `updated_at` is automatically set to the current timestamp on every UPDATE — no application code needed.

---

## Summary

| Concept | Key takeaway |
|---------|-------------|
| **Persistence** | Data survives across sessions — the core reason databases exist |
| **Disk vs RAM** | Databases use disk (cheap, large); caching uses RAM (fast, expensive) |
| **Relational vs NoSQL** | PostgreSQL for structured data + integrity; MongoDB for flexible/prototyping |
| **PostgreSQL** | First choice — open source, SQL-compliant, great JSON support |
| **Data types** | Use `text` over `varchar`, `decimal` for money, `jsonb` over `json`, `uuid` for PKs |
| **Migrations** | Version-controlled SQL files — up (apply) and down (rollback) |
| **Constraints** | NOT NULL on most fields, UNIQUE on emails, CHECK for ranges, enums for allowed values |
| **Relationships** | 1:1 (shared PK), 1:many (FK in "many" table), many:many (linking table + composite PK) |
| **Referential integrity** | ON DELETE RESTRICT/CASCADE/SET NULL to protect related data |
| **Parameterized queries** | Always use parameters, never string concatenation — prevents SQL injection |
| **Indexes** | Create on fields in JOINs, WHERE, ORDER BY — speeds reads, costs writes |
| **Triggers** | Automate operations (e.g. updating `updated_at`) at the database level |
