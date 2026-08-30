# BookMyShow — System Design (End to End)

---

## Step 1: Functional Requirements

Think like an end user walking through the app.

1. Browse movies and theaters
2. Select seats
3. Book tickets
4. Payment
5. Cancellation / refund

---

## Step 2: Estimate Scale

- **DAU:** 10M (1% of India's 1.4B population)
- **Avg req/sec:** 10M / 86,400 = ~115 req/sec
- **Peak req/sec:** 115 x 10 = ~1,150 req/sec (new movie release)
- **Storage:** 1M bookings/day x 1KB = 1GB/day → ~365GB/year

---

## Step 3: Core Entities

1. User
2. Movie
3. Theater
4. Screen
5. Showtime
6. Seat
7. Booking
8. Payment

**Relationships:**

- Theater → has many → Screens
- Screen → has many → Seats
- Movie + Screen → Showtime
- User → makes → Booking
- Booking → has many → Seats

---

## Step 4: High-Level Architecture

Client (Web / Mobile)
↓
API Gateway
(Auth | Routing)
↓
┌──────────────────────────────────┐
│ Movie │ Theater │ Booking │
│ Service │ Service │ Service │
│ │ Payment │
│ │ Service │
└──────────────────────────────────┘
↓ ↓ ↓
PostgreSQL Redis Kafka
(bookings, (cache) (booking
payments) queue)

- **API Gateway** — authentication, routing, rate limiting
- **PostgreSQL** — relational data (bookings, payments)
- **Redis** — caching movie/theater data, seat locking
- **Kafka** — queuing booking requests during traffic spikes

---

## Step 5: Deep Dive — Hardest Problem (Double Booking)

**Problem:** Two users select the same seat simultaneously.

**Solution: Redis SET NX + TTL**

Flow:

1. User selects seat → Redis SET NX locks it for 10 min
2. Second user tries same seat → SET NX returns False → blocked
3. User completes payment → seat marked BOOKED in PostgreSQL → Redis lock deleted
4. User abandons → TTL expires → seat automatically available again

**Why SET NX?** It's atomic — only one user gets the lock even if thousands hit simultaneously.

---

## Step 6: Non-Functional Requirements

**High Availability:**

- Load balancer + multiple service instances
- DB failover (primary goes down → promote replica)

**Scalability:**

- Horizontal scaling for stateless services
- Kafka to queue booking requests during peak traffic

**Performance:**

- Redis cache for movie/theater browsing (read-heavy, rarely changes)
- DB indexes on showtime, seat, booking tables

**Consistency:**

- Redis SET NX ensures no double booking
- ACID transactions in PostgreSQL for payments

**Cache Freshness:**

- TTL on cached data
- Delete cache on update (Cache-Aside pattern)

---

## Step 7: Trade-offs

**Redis for seat locking:**

> "I chose Redis because it's fast, atomic with SET NX, and TTL handles
> abandoned bookings automatically. The trade-off is if Redis crashes,
> seat locks are lost. I mitigate this with Redis Sentinel / Redis Cluster
> for high availability."

**PostgreSQL for bookings:**

> "I chose PostgreSQL because our data is relational and we need ACID
> transactions for bookings and payments. The trade-off is it's hard to
> scale horizontally compared to NoSQL. I mitigate this with read replicas
> for read-heavy traffic and connection pooling to manage DB connections."

**Kafka for peak traffic:**

> "I chose Kafka to queue booking requests during traffic spikes so services
> aren't overwhelmed. The trade-off is operational complexity. I mitigate
> this by using a managed service like Confluent."

---

## Areas to Remember

- Core entities — "Booking" is the key entity that ties User + Showtime + Seats together
- Trade-off formula — "I chose X because Y. Trade-off is Z. I mitigate by W."
- Always justify your numbers in scale estimation
- Deep dive = state the problem → naive solution → why it fails → your fix
