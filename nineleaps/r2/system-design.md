# System Design / LLD — Round 2 Prep (Uber Client Round)

---

## 1. Kafka / Queuing Systems

### What Problem Does It Solve?
In a distributed system, services need to communicate. Direct calls (REST) create tight coupling:
- If Service B is slow → Service A is blocked
- If Service B is down → Service A fails

A message queue decouples them. Service A drops a message and moves on. Service B processes it when ready.

---

### Kafka vs Simple Queues (Redis/RabbitMQ)

| | Redis Queue | RabbitMQ | Kafka |
|---|---|---|---|
| Use case | Simple tasks | Complex routing | High-throughput event streaming |
| Message retention | Deleted after consumed | Deleted after consumed | Retained (configurable) |
| Throughput | Low-Medium | Medium | Very High (millions/sec) |
| Replay messages | No | No | Yes |
| Best for | Background jobs | Microservice messaging | Real-time events, logs, analytics |

---

### Kafka Core Concepts

```
Producer → [Topic: user-events] → Consumer Group
                 |
           Partition 1: [msg1, msg4, msg7]
           Partition 2: [msg2, msg5, msg8]
           Partition 3: [msg3, msg6, msg9]
```

- **Producer** — sends messages to a topic
- **Topic** — a named stream of messages (like a table in a DB)
- **Partition** — topics are split into partitions for parallelism
- **Consumer** — reads messages from a topic
- **Consumer Group** — multiple consumers sharing the load; each partition is read by one consumer
- **Offset** — position of a message in a partition; consumers track their offset
- **Broker** — a Kafka server; a cluster has multiple brokers

---

### Why Kafka for Real-Time Event Sync?
- Messages are **retained** — consumers can replay events
- **High throughput** — handles millions of events per second
- **Multiple consumers** can read the same topic independently
- **Fault tolerant** — messages are replicated across brokers

**Example at Uber:** Driver location updates every 4 seconds. Kafka streams these events to:
- Trip service (track driver)
- ETA service (recalculate arrival time)
- Surge pricing service (demand analysis)
- Analytics (dashboards)

All read from the same topic independently.

---

### Kafka vs Queuing (When to Use What)

**Use a Queue (Celery + Redis / RabbitMQ) when:**
- Task needs to run once (send email, resize image)
- Order doesn't strictly matter
- Simple producer-consumer

**Use Kafka when:**
- Multiple services need the same event
- Need to replay events
- High throughput (millions of events/sec)
- Event sourcing / audit log
- Real-time streaming (location tracking, analytics)

---

### Simple Python Kafka Example
```python
from kafka import KafkaProducer, KafkaConsumer
import json

# Producer
producer = KafkaProducer(
    bootstrap_servers='localhost:9092',
    value_serializer=lambda v: json.dumps(v).encode('utf-8')
)
producer.send('driver-location', {'driver_id': 'D123', 'lat': 12.9, 'lng': 77.6})

# Consumer
consumer = KafkaConsumer(
    'driver-location',
    bootstrap_servers='localhost:9092',
    value_deserializer=lambda v: json.loads(v.decode('utf-8'))
)
for message in consumer:
    print(message.value)  # {'driver_id': 'D123', 'lat': 12.9, 'lng': 77.6}
```

---

## 2. Redis — Caching

### What is Redis?
Redis is an in-memory key-value store. Extremely fast (microsecond reads) because data lives in RAM.

Used for: caching, session storage, rate limiting, queues, pub/sub.

---

### Caching Strategies

**Cache-Aside (most common):**
```
Read:  check cache → hit → return | miss → read DB → write cache → return
Write: write DB → delete/update cache
```

**Write-Through:** write to cache + DB simultaneously. Cache always fresh. Slower writes.

**Write-Behind:** write to cache first, DB updated async. Fast writes, risk of data loss.

**Read-Through:** cache fetches from DB automatically on miss. App only talks to cache.

---

### Redis Data Structures

| Structure | Use Case |
|---|---|
| String | Simple key-value, counters, session tokens |
| Hash | Store object fields (user profile) |
| List | Queues, activity feeds |
| Set | Unique items, tags |
| Sorted Set | Leaderboards, rate limiting (sliding window) |
| TTL (on any key) | Auto-expiry for cache invalidation |

---

### Common Redis Patterns

**Cache a DB query:**
```python
import redis
import json

r = redis.Redis(host='localhost', port=6379)

def get_user(user_id):
    cache_key = f"user:{user_id}"
    cached = r.get(cache_key)

    if cached:
        return json.loads(cached)

    user = db.query(f"SELECT * FROM users WHERE id={user_id}")
    r.setex(cache_key, 3600, json.dumps(user))  # TTL = 1 hour
    return user
```

**Session storage:**
```python
r.setex(f"session:{token}", 86400, user_id)  # expires in 24 hours
```

**Rate limiting (sliding window — from Round 1):**
```python
# Sorted Set: score = timestamp, member = request id
r.zremrangebyscore(key, 0, now - window)
r.zadd(key, {str(now): now})
count = r.zcard(key)
```

---

### Cache Invalidation — The Hard Problem
When DB data changes, cache must be updated/deleted. Three strategies:
1. **TTL** — cache expires after N seconds. Simple but stale data possible.
2. **Delete on write** — when DB is updated, delete cache key. Next read repopulates.
3. **Write-through** — update cache and DB together.

---

### When to Use Redis
- Frequently read, rarely changed data (user profiles, product details)
- Session/token storage
- Rate limiting
- Leaderboards (sorted sets)
- Real-time pub/sub (live notifications)
- Expensive computation results (API responses, ML predictions)

---

## 3. SQL vs NoSQL — When to Use Which

### SQL (Relational Databases)
Examples: PostgreSQL, MySQL

- Data stored in **tables with rows and columns**
- **Schema is fixed** — define structure upfront
- **ACID** transactions (Atomicity, Consistency, Isolation, Durability)
- Supports **JOINs** — relationships between tables
- Scales **vertically** (bigger machine)

---

### NoSQL (Non-Relational Databases)
Examples: MongoDB (document), Redis (key-value), Cassandra (column), Neo4j (graph)

- Flexible / schema-less
- Scales **horizontally** (add more machines)
- Eventual consistency (not always ACID)
- No JOINs — denormalized data

---

### When to Use SQL
- Data has clear relationships (users → orders → products)
- Need transactions (banking, payments, bookings)
- Need complex queries with JOINs and aggregations
- Data structure is well-defined and stable
- Consistency is critical

**Examples:** BookMyShow (seat booking), banking, e-commerce orders

---

### When to Use NoSQL

| Type | Use Case | Example |
|---|---|---|
| Document (MongoDB) | Flexible schema, nested data | Product catalog, CMS |
| Key-Value (Redis) | Caching, sessions, rate limiting | Session store |
| Column (Cassandra) | Time-series, write-heavy, huge scale | Uber trip logs, IoT |
| Graph (Neo4j) | Relationships (friends, recommendations) | Social network |

---

### Decision Framework

```
Is data highly relational?          → SQL
Need ACID transactions?             → SQL
Schema changes frequently?          → NoSQL (Document)
Need massive write throughput?      → NoSQL (Cassandra)
Need horizontal scaling?            → NoSQL
Caching / sessions?                 → Redis
Huge scale + time-series data?      → Cassandra
```

---

### At Uber (Real World)
- **PostgreSQL** — trips, payments, user accounts (relational, ACID)
- **Cassandra** — driver location history, trip events (massive writes, time-series)
- **Redis** — caching, rate limiting, session storage
- **Kafka** — event streaming between services

---

## 4. Design BookMyShow

### Requirements
**Functional:**
- Browse movies, cities, theaters, showtimes
- Select seats and book tickets
- Payment
- Booking confirmation

**Non-Functional:**
- High availability
- No double booking (consistency for seat selection)
- Handle traffic spikes (new movie release)

---

### Core Entities
```
User, Movie, Theater, Screen, Showtime, Seat, Booking, Payment
```

---

### High-Level Architecture

```
Client (Web/App)
    ↓
API Gateway
    ↓
┌─────────────────────────────────────┐
│  Movie Service  │  Booking Service  │
│  Theater Service│  Payment Service  │
│  Search Service │  Notification Svc │
└─────────────────────────────────────┘
    ↓               ↓
PostgreSQL        Redis (seat locks)
                  Kafka (events)
```

---

### Critical Problem: Seat Booking Race Condition
Two users try to book the same seat simultaneously.

**Solution: Temporary Seat Lock**
1. User selects seat → lock it in Redis for 10 minutes
2. User completes payment → mark seat as BOOKED in DB
3. If user abandons → Redis TTL expires → seat becomes available again

```python
def lock_seat(show_id, seat_id, user_id):
    key = f"seat_lock:{show_id}:{seat_id}"
    # NX = only set if key doesn't exist (atomic)
    locked = r.set(key, user_id, ex=600, nx=True)
    return locked  # True if locked, False if already taken

def confirm_booking(show_id, seat_id, user_id):
    key = f"seat_lock:{show_id}:{seat_id}"
    if r.get(key) == user_id:
        # update DB: seat status = BOOKED
        db.execute("UPDATE seats SET status='BOOKED' WHERE ...")
        r.delete(key)
        return True
    return False
```

---

### Database Schema (Simplified)

```sql
CREATE TABLE movies (
    id          SERIAL PRIMARY KEY,
    title       VARCHAR(255),
    duration    INT,
    language    VARCHAR(50),
    genre       VARCHAR(100)
);

CREATE TABLE theaters (
    id          SERIAL PRIMARY KEY,
    name        VARCHAR(255),
    city        VARCHAR(100),
    location    VARCHAR(255)
);

CREATE TABLE screens (
    id          SERIAL PRIMARY KEY,
    theater_id  INT REFERENCES theaters(id),
    name        VARCHAR(50),
    total_seats INT
);

CREATE TABLE showtimes (
    id          SERIAL PRIMARY KEY,
    movie_id    INT REFERENCES movies(id),
    screen_id   INT REFERENCES screens(id),
    start_time  TIMESTAMP,
    end_time    TIMESTAMP
);

CREATE TABLE seats (
    id          SERIAL PRIMARY KEY,
    screen_id   INT REFERENCES screens(id),
    row         VARCHAR(5),
    number      INT,
    type        VARCHAR(20)  -- regular, premium, recliner
);

CREATE TABLE bookings (
    id          SERIAL PRIMARY KEY,
    user_id     INT REFERENCES users(id),
    showtime_id INT REFERENCES showtimes(id),
    status      VARCHAR(20),  -- pending, confirmed, cancelled
    total_price DECIMAL,
    booked_at   TIMESTAMP
);

CREATE TABLE booking_seats (
    booking_id  INT REFERENCES bookings(id),
    seat_id     INT REFERENCES seats(id),
    PRIMARY KEY (booking_id, seat_id)
);
```

---

### Handling Traffic Spikes (New Movie Release)
- **Cache** movie/theater/showtime data in Redis (read-heavy, rarely changes)
- **Queue** booking requests during peak (Kafka) — process in order
- **CDN** for static assets
- **Read replicas** for DB — browse queries go to replicas, writes go to primary

---

## 5. DB Normalization & Data Modeling

### What is Normalization?
Organizing tables to reduce **data redundancy** and improve **data integrity.**

---

### Normal Forms

**1NF (First Normal Form):**
- Each column has atomic (single) values
- No repeating groups

```
BAD:
| user_id | phone_numbers        |
|---------|----------------------|
| 1       | 9999, 8888           |  ← not atomic

GOOD:
| user_id | phone_number |
|---------|--------------|
| 1       | 9999         |
| 1       | 8888         |
```

---

**2NF (Second Normal Form):**
- Must be in 1NF
- Every non-key column depends on the **entire** primary key (no partial dependency)

```
BAD (composite key: order_id + product_id):
| order_id | product_id | product_name | quantity |
product_name depends only on product_id, not the full key

GOOD: split into two tables:
orders_products(order_id, product_id, quantity)
products(product_id, product_name)
```

---

**3NF (Third Normal Form):**
- Must be in 2NF
- No **transitive dependencies** (non-key column depends on another non-key column)

```
BAD:
| student_id | zip_code | city     |
city depends on zip_code, not student_id

GOOD: split:
students(student_id, zip_code)
zip_codes(zip_code, city)
```

---

### Denormalization — When to Break the Rules
Sometimes normalization hurts read performance (too many JOINs). You denormalize intentionally:

- Store `city` directly in the user table (avoids a JOIN)
- Store `total_price` in orders (avoids recalculating each time)

**Rule:** Normalize first, denormalize only when you have a proven performance problem.

---

### Data Modeling Best Practices
- Use surrogate keys (auto-increment `id`) as primary keys
- Use foreign keys to enforce relationships
- Index columns used in WHERE, JOIN, ORDER BY
- Use `ENUM` or lookup tables for fixed values (status, type)
- Always store timestamps: `created_at`, `updated_at`
- Soft delete with `deleted_at` instead of actually deleting rows

---

## 6. Table Joins — Available Theaters in a Location

### The Query
"Show me all theaters in Bangalore showing Movie X today."

---

### Schema Involved
```
movies → showtimes → screens → theaters
```

---

### SQL Query

```sql
SELECT DISTINCT
    t.id          AS theater_id,
    t.name        AS theater_name,
    t.location,
    s.start_time,
    sc.name       AS screen_name
FROM theaters t
JOIN screens sc       ON sc.theater_id = t.id
JOIN showtimes s      ON s.screen_id = sc.id
JOIN movies m         ON m.id = s.movie_id
WHERE
    t.city = 'Bangalore'
    AND m.title = 'Inception'
    AND DATE(s.start_time) = CURRENT_DATE
    AND s.start_time > NOW()           -- only future shows
ORDER BY s.start_time;
```

---

### Types of JOINs

```
INNER JOIN  — only rows that match in both tables
LEFT JOIN   — all rows from left, matching rows from right (NULL if no match)
RIGHT JOIN  — all rows from right, matching rows from left
FULL JOIN   — all rows from both, NULL where no match
```

**Example where LEFT JOIN matters:**
"Show all theaters, even those with no shows today."

```sql
SELECT t.name, s.start_time
FROM theaters t
LEFT JOIN screens sc   ON sc.theater_id = t.id
LEFT JOIN showtimes s  ON s.screen_id = sc.id
    AND DATE(s.start_time) = CURRENT_DATE
WHERE t.city = 'Bangalore';
-- theaters with no shows will appear with NULL start_time
```

---

### Indexing for Performance
This query runs slow at scale without indexes.

```sql
CREATE INDEX idx_theaters_city        ON theaters(city);
CREATE INDEX idx_showtimes_screen     ON showtimes(screen_id);
CREATE INDEX idx_showtimes_start_time ON showtimes(start_time);
CREATE INDEX idx_showtimes_movie      ON showtimes(movie_id);
CREATE INDEX idx_screens_theater      ON screens(theater_id);
```

---

### Available Seats for a Showtime
"How many seats are available for show ID 42?"

```sql
SELECT
    COUNT(*) FILTER (WHERE bs.seat_id IS NULL) AS available_seats,
    COUNT(*) AS total_seats
FROM seats s
JOIN screens sc ON sc.id = s.screen_id
JOIN showtimes sh ON sh.screen_id = sc.id
LEFT JOIN booking_seats bs ON bs.seat_id = s.id
    JOIN bookings b ON b.id = bs.booking_id
        AND b.showtime_id = 42
        AND b.status = 'confirmed'
WHERE sh.id = 42;
```

Or simpler — track seat status directly:

```sql
-- Add status column to seats per showtime
CREATE TABLE showtime_seats (
    showtime_id INT REFERENCES showtimes(id),
    seat_id     INT REFERENCES seats(id),
    status      VARCHAR(20) DEFAULT 'available',  -- available, locked, booked
    PRIMARY KEY (showtime_id, seat_id)
);

-- Query available seats
SELECT COUNT(*) FROM showtime_seats
WHERE showtime_id = 42 AND status = 'available';
```

---

## Quick Reference — When to Use What

| Scenario | Solution |
|---|---|
| Real-time events to multiple services | Kafka |
| Background jobs (email, resize) | Celery + Redis |
| Cache DB queries | Redis (Cache-Aside) |
| Seat booking race condition | Redis lock (SET NX) |
| Financial transactions | SQL (ACID) |
| Huge write throughput, time-series | Cassandra (NoSQL) |
| Flexible schema | MongoDB (NoSQL) |
| Complex queries with relationships | PostgreSQL (SQL) |
| Reduce duplicate data in DB | Normalize (3NF) |
| Improve read performance | Denormalize + Indexes |
