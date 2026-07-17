# System Design Interview Prep — Basic to Advanced (11 YOE)

## Why this matters more at your level
At 11 YOE, system design typically carries **more weight than DSA**. Interviewers are checking: can you make sound tradeoffs, structure ambiguity, and explain *why* — not just name-drop buzzwords like "use Kafka" or "add a cache." Every Q&A below focuses on the **why**, because that's what actually gets probed in the room.

---

## 8-Week Roadmap

| Week | Focus |
|---|---|
| 1 | Fundamentals: scalability, load balancing, caching |
| 2 | Databases: SQL vs NoSQL, indexing, replication, sharding |
| 3 | CAP theorem, consistency models, consistent hashing |
| 4 | Messaging: queues, pub-sub, async processing |
| 5 | Microservices, API design, rate limiting, idempotency |
| 6 | Advanced: consensus, distributed transactions, CQRS/event sourcing |
| 7 | Practice HLD+LLD problems (2-3 full mocks) |
| 8 | Mock interviews + refine communication/tradeoff articulation |

**How to answer in the interview:** Requirements → Scale estimates (back-of-envelope) → High-Level Design (boxes) → Deep-dive on 1-2 components → Data model → Bottlenecks & tradeoffs. Always state assumptions out loud.

---

# PART 1: FOUNDATIONAL CONCEPTS (Q&A)

### Q: What's the difference between vertical and horizontal scaling?
**Why asked:** Tests if you understand scaling limits and when each applies.
**A:** Vertical scaling = adding more power (CPU/RAM) to one machine — simple but has a hard ceiling and single point of failure. Horizontal scaling = adding more machines — more complex (needs load balancing, data partitioning) but scales near-linearly and improves fault tolerance. Interviewers want you to default to horizontal for anything at real product scale.

### Q: How does a load balancer decide which server to send traffic to?
**Why asked:** Checks if you know it's not "magic" — algorithms have tradeoffs.
**A:** Common algorithms: Round Robin (simple, ignores load), Least Connections (better for uneven request durations), IP Hash (same client → same server, useful for session stickiness), Weighted variants (accounts for server capacity differences). Layer 4 (transport, fast, less smart) vs Layer 7 (application, can route by URL/header, slightly slower).

### Q: Why use a cache, and what problems can it introduce?
**Why asked:** Everyone says "add a cache" — interviewer wants to know if you understand the failure modes.
**A:** Caching reduces latency and DB load by serving hot data from memory. Problems it introduces: **stale data** (needs invalidation strategy), **cache stampede** (many requests miss simultaneously and hammer the DB — mitigated with locks or request coalescing), **cold start** (empty cache after restart). Eviction policies: LRU (most common), LFU, FIFO. Strategies: cache-aside (app manages it, most common), write-through (write to cache and DB together, consistent but slower writes), write-back (write to cache first, async to DB, fast but risk of data loss).

### Q: SQL vs NoSQL — how do you choose?
**Why asked:** Very common; tests if you pick based on actual requirements, not trend.
**A:** SQL (Postgres/MySQL): strong consistency, relational integrity, complex joins/transactions — choose when data is structured and relationships matter (e.g., banking, orders). NoSQL: 
- **Document (MongoDB):** flexible schema, good for nested/varying data.
- **Key-Value (Redis, DynamoDB):** simplest, fastest, great for caching/sessions.
- **Wide-column (Cassandra):** massive write throughput, time-series/event data.
- **Graph (Neo4j):** relationship-heavy data (social networks, recommendations).
Choose NoSQL when you need horizontal scale, flexible schema, or specific access patterns over strict consistency.

### Q: What is database indexing, and what's the tradeoff?
**Why asked:** Checks real understanding vs "indexes make things faster" cliché.
**A:** An index (typically a B-Tree or hash structure) lets the DB find rows without scanning the whole table — turns O(n) lookups into O(log n). Tradeoff: every index **slows down writes** (insert/update/delete must update the index too) and **uses extra storage**. So you index columns that are frequently queried/filtered/joined on, not everything.

### Q: What is database replication, and what problems does it solve/create?
**Why asked:** Every real system replicates data — interviewer wants to see you understand consistency lag.
**A:** Replication = keeping copies of data on multiple nodes (leader-follower/master-slave common pattern). Solves: read scaling (reads from replicas), fault tolerance (failover if leader dies). Creates: **replication lag** (replicas can serve stale data momentarily) — this is why "read-your-own-write" issues happen, solved with read-from-leader-for-critical-reads or session stickiness.

### Q: What is database sharding, and how do you pick a shard key?
**Why asked:** Tests whether you understand this is the hardest part of scaling writes.
**A:** Sharding = splitting data across multiple databases so no single DB holds everything (scales writes, unlike replication). Shard key choice is critical: a bad key (e.g., sequential IDs) causes **hotspotting** (all new writes go to one shard). Good keys distribute load evenly — often a hash of user ID, or composite keys. Downsides: cross-shard joins/transactions become hard, and resharding later is painful (mitigated with consistent hashing).

### Q: Explain the CAP theorem in your own words.
**Why asked:** Extremely common; interviewer wants a practical explanation, not textbook recitation.
**A:** In a distributed system, during a network partition, you must choose between **Consistency** (every read gets the latest write) and **Availability** (every request gets a response, possibly stale). You can't have both during a partition. Real systems pick based on use case: banking leans CP (reject requests rather than show wrong balance), social media feeds lean AP (show slightly stale data rather than go down).

### Q: What is consistent hashing and why not just use `hash(key) % N`?
**Why asked:** Classic "do you actually understand distributed systems" question.
**A:** With `hash(key) % N`, adding/removing a server changes N, which **remaps almost all keys** — causing massive cache misses or data movement. Consistent hashing places servers and keys on a hash ring; when a server is added/removed, only the keys in its immediate ring segment move — roughly `1/N` of keys, not all of them. Virtual nodes are added to avoid uneven load distribution.

### Q: When would you use a message queue, and what's the tradeoff vs a direct API call?
**Why asked:** Tests understanding of sync vs async architecture decisions.
**A:** Use a queue (Kafka/RabbitMQ/SQS) when the producer shouldn't block on the consumer's processing time, when you need to smooth traffic spikes (buffer), or decouple services. Tradeoff: added complexity (message ordering, at-least-once vs exactly-once delivery, dead-letter queues for failures), and eventual rather than immediate consistency.

### Q: What's the difference between a monolith and microservices, and when do you actually need microservices?
**Why asked:** Senior candidates are expected to push back on "microservices by default."
**A:** Monolith: one deployable unit, simpler to develop/deploy/debug initially, but scaling and team ownership get harder as it grows. Microservices: independent services, independently scalable/deployable, but adds network latency, distributed debugging complexity, and data consistency challenges across services. **You need microservices when team size and scaling needs justify the operational overhead — not by default.** Many "senior" answers actually favor starting with a modular monolith.

### Q: What is idempotency and why does it matter in API design?
**Why asked:** Very practical — comes up constantly in payment/order systems.
**A:** An idempotent operation produces the same result no matter how many times it's executed (e.g., "set balance to $100" vs "add $10" — the former is idempotent, the latter isn't). Matters because networks are unreliable — clients retry requests, and without idempotency, a retried "charge card" request could double-charge a customer. Solved with idempotency keys: client sends a unique key, server tracks which keys were already processed.

---

# PART 2: ADVANCED CONCEPTS (Q&A)

### Q: Explain the difference between strong, eventual, and causal consistency.
**Why asked:** Tests depth beyond CAP theorem buzzwords.
**A:** Strong consistency: all reads reflect the latest write immediately (simple to reason about, costs latency/availability). Eventual consistency: replicas converge "eventually" if no new writes occur — reads may return stale data momentarily (used in DNS, many NoSQL stores). Causal consistency: middle ground — operations that are causally related (like a comment after a post) are seen in order by all nodes, but unrelated operations can be seen in any order.

### Q: What are 2PC (Two-Phase Commit) and Saga, and when do you use each?
**Why asked:** Distributed transactions are a classic "senior" differentiator question.
**A:** 2PC: a coordinator asks all participants to "prepare" (lock resources), then commits only if all agree — strongly consistent but blocks if the coordinator or a participant fails (poor availability, not partition-tolerant). Saga: break a transaction into a sequence of local transactions, each with a compensating action if a later step fails (e.g., refund if shipping fails) — better availability, used in microservices, but you must design compensations carefully and accept temporary inconsistency.

### Q: What problem does distributed consensus (Raft/Paxos) solve, at a high level?
**Why asked:** You're not expected to implement Raft, but you should know why it exists.
**A:** It solves how multiple nodes agree on a single value/order of operations despite failures and network issues — the foundation for leader election and replicated logs (used in etcd, Kafka controller election, distributed databases). Key idea (Raft, simpler to explain than Paxos): nodes elect a leader via majority vote; the leader replicates log entries to followers; an entry is "committed" once a majority acknowledge it — tolerates minority node failures.

### Q: What is CQRS and event sourcing, and what problem do they solve?
**Why asked:** Shows exposure to modern architecture patterns for complex domains.
**A:** CQRS (Command Query Responsibility Segregation): separate models for writes (commands) and reads (queries) — lets you optimize/scale each independently (e.g., normalized write DB, denormalized read cache). Event Sourcing: instead of storing current state, store the sequence of events that led to it — state is derived by replaying events. Together they give full audit history and flexible read models, at the cost of complexity and eventual consistency between write and read sides.

### Q: How do you design for rate limiting, and what algorithms exist?
**Why asked:** Practical, commonly asked as its own mini system design.
**A:** Algorithms: **Token Bucket** (tokens refill at fixed rate, request consumes a token — allows bursts up to bucket size), **Leaky Bucket** (processes requests at fixed rate, smooths bursts), **Fixed Window Counter** (simple but allows 2x burst at window boundaries), **Sliding Window Log/Counter** (more accurate, more memory). At scale, rate limit state is kept in a fast shared store (Redis) so it works across multiple servers, not per-instance.

### Q: How would you design for observability in a distributed system?
**Why asked:** Senior engineers are expected to think about operability, not just features.
**A:** Three pillars: **Metrics** (aggregated numeric data — latency percentiles, error rates, via Prometheus/Grafana), **Logging** (structured, centralized — ELK/Loki), **Distributed Tracing** (follow a single request across services — Jaeger/Zipkin, using trace IDs propagated through headers). Without tracing, debugging a slow request across 10 microservices is nearly impossible.

### Q: What's a distributed lock, and why is it hard to get right?
**Why asked:** Common follow-up when discussing race conditions in distributed systems.
**A:** A distributed lock (e.g., via Redis with Redlock, or Zookeeper) ensures only one process across multiple machines can do something at a time (like a scheduled job). It's hard because: locks can expire while the holder is still working (need fencing tokens to prevent stale lock holders from acting), and the lock service itself can fail or partition. Simpler alternative when possible: design for idempotency instead of locking.

---

# PART 3: SAMPLE PRACTICE QUESTIONS (HLD + LLD + Diagrams)

## Practice 1: Design a URL Shortener (like bit.ly)

**Requirements:** Shorten a long URL, redirect short → long, handle ~100M new URLs/day, reads >> writes (100:1).

### High-Level Design (Architecture)
```
                    ┌─────────────┐
   Client  ───────► │ Load Balancer│
                    └──────┬──────┘
                           │
              ┌────────────┴────────────┐
              ▼                         ▼
        ┌───────────┐            ┌───────────┐
        │  App Server│            │ App Server│  (stateless, horizontally scaled)
        └─────┬──────┘            └─────┬─────┘
              │                         │
      ┌───────┴────────┐        ┌───────┴───────┐
      ▼                ▼        ▼               ▼
 ┌─────────┐    ┌────────────┐          ┌──────────────┐
 │  Redis   │    │  ID        │          │  DB (sharded) │
 │  Cache   │    │  Generator │          │  short→long   │
 │(hot URLs)│    │ (counter/  │          │  mapping      │
 └─────────┘    │  Snowflake)│          └──────────────┘
                 └────────────┘
```
**Flow:** Write: client submits long URL → app server requests a unique ID from ID generator → encode ID to base62 → store mapping in DB → return short URL. Read: client hits short URL → check cache first → fallback to DB → 301/302 redirect.

**Key design decisions & why:**
- **ID generation:** Base62 encode an auto-incrementing/Snowflake-style distributed ID (not hash of URL — avoids collision handling entirely). Snowflake gives unique IDs across multiple servers without coordination.
- **Cache:** Since reads >> writes, a cache in front of the DB absorbs most read traffic for popular links.
- **DB choice:** Simple key-value access pattern → a key-value store (DynamoDB) or sharded SQL both work; shard by hash of the short code for even distribution.
- **Redirect type:** 302 (temporary) if you want to track analytics on every click; 301 (permanent) if you want browsers to cache and reduce your server load.

### Low-Level Design (Class Diagram)
```
┌────────────────────┐        ┌──────────────────────┐
│   UrlShortenerService│──────►│   IdGenerator (interface)│
├────────────────────┤        ├──────────────────────┤
│ + shorten(longUrl)  │        │ + generateId(): long  │
│ + resolve(shortCode)│        └──────────┬───────────┘
└─────────┬───────────┘                   │
          │                     ┌─────────┴─────────┐
          ▼                     ▼                   ▼
┌────────────────────┐  ┌───────────────┐   ┌────────────────┐
│  UrlRepository      │  │SnowflakeIdGen  │   │CounterIdGen    │
│  (interface)        │  └───────────────┘   └────────────────┘
├────────────────────┤
│ + save(UrlMapping)  │
│ + findByCode(code)  │
└─────────┬───────────┘
          ▼
┌────────────────────┐
│   UrlMapping (entity)│
├────────────────────┤
│ - shortCode: String │
│ - longUrl: String    │
│ - createdAt: Instant │
│ - expiresAt: Instant │
└────────────────────┘
```
```java
interface IdGenerator {
    long generateId();
}

class Base62Encoder {
    private static final String ALPHABET =
        "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz";

    public String encode(long id) {
        StringBuilder sb = new StringBuilder();
        while (id > 0) {
            sb.append(ALPHABET.charAt((int) (id % 62)));
            id /= 62;
        }
        return sb.reverse().toString();
    }
}

class UrlShortenerService {
    private final IdGenerator idGenerator;
    private final Base62Encoder encoder;
    private final UrlRepository repository;
    private final Cache cache;

    public String shorten(String longUrl) {
        long id = idGenerator.generateId();
        String shortCode = encoder.encode(id);
        repository.save(new UrlMapping(shortCode, longUrl, Instant.now()));
        return shortCode;
    }

    public String resolve(String shortCode) {
        String cached = cache.get(shortCode);
        if (cached != null) return cached;
        UrlMapping mapping = repository.findByCode(shortCode);
        if (mapping == null) throw new NotFoundException();
        cache.put(shortCode, mapping.getLongUrl());
        return mapping.getLongUrl();
    }
}
```

---

## Practice 2: Design a Rate Limiter (as a service)

**Requirements:** Limit requests per user/API key (e.g., 100 req/min), work across multiple servers, low latency.

### High-Level Design (Architecture)
```
   Client Request
        │
        ▼
 ┌─────────────┐        ┌────────────────────┐
 │ API Gateway  │───────►│ Rate Limiter Service│
 │ (or filter)  │        └──────────┬─────────┘
 └──────┬───────┘                   │
        │ (if allowed)              ▼
        ▼                   ┌───────────────┐
 ┌─────────────┐            │ Redis (shared  │
 │ Backend      │            │ counters, TTL) │
 │ Service      │            └───────────────┘
 └─────────────┘
```
**Why Redis:** rate limit counters must be shared across all app server instances — in-memory per-server counters would let a user get `N × server_count` requests instead of `N`. Redis gives atomic increment + TTL in one round trip (`INCR` + `EXPIRE`, or a Lua script for atomicity).

**Algorithm choice & why:** Token Bucket is usually preferred in interviews — it allows short bursts (good UX) while enforcing an average rate, and is simple to reason about (refill rate + bucket size).

### Low-Level Design (Class Diagram)
```
┌─────────────────────┐
│  RateLimiter (interface)│
├─────────────────────┤
│ + allowRequest(key): boolean │
└──────────┬───────────┘
           │
   ┌───────┴────────┐
   ▼                ▼
┌────────────────┐ ┌────────────────────┐
│TokenBucketLimiter│ │SlidingWindowLimiter│
└────────────────┘ └────────────────────┘
           │
           ▼
┌─────────────────────┐
│  RedisTokenBucketStore│
├─────────────────────┤
│ + getTokens(key)      │
│ + setTokens(key, val) │
└─────────────────────┘
```
```java
interface RateLimiter {
    boolean allowRequest(String key);
}

class TokenBucketLimiter implements RateLimiter {
    private final int capacity;
    private final double refillRatePerSec;
    private final RedisClient redis;

    public TokenBucketLimiter(int capacity, double refillRatePerSec, RedisClient redis) {
        this.capacity = capacity;
        this.refillRatePerSec = refillRatePerSec;
        this.redis = redis;
    }

    @Override
    public boolean allowRequest(String key) {
        // Atomic Lua script executed in Redis to avoid race conditions:
        // 1. Read current tokens + last refill timestamp
        // 2. Compute tokens to add = elapsed_seconds * refillRatePerSec
        // 3. new_tokens = min(capacity, current_tokens + tokens_to_add)
        // 4. if new_tokens >= 1: consume one, allow; else: deny
        String luaScript = "-- token bucket logic executed atomically in Redis --";
        Long allowed = redis.eval(luaScript, key, capacity, refillRatePerSec);
        return allowed == 1;
    }
}
```

---

## Practice 3: Design a Parking Lot System (classic LLD-heavy question)

**Requirements:** Multiple floors, multiple spot types (compact/large/handicapped), track availability, calculate fees on exit.

### High-Level Architecture
```
┌────────────┐     ┌──────────────────┐     ┌───────────────┐
│ Entry Gate  │────►│ ParkingLotService │────►│  Payment/Exit  │
│ (ticket gen)│     │  (spot allocation)│     │  Gate          │
└────────────┘     └────────┬─────────┘     └───────────────┘
                             ▼
                    ┌──────────────────┐
                    │ In-memory/DB store│
                    │ of floors & spots │
                    └──────────────────┘
```
This is a single-process/single-facility system in most interviews — the depth expected here is in the **class design**, not distributed architecture (though you can mention a DB-backed spot registry for persistence/multi-location scale).

### Class Diagram
```
┌────────────────┐
│ ParkingLot      │ (singleton)
├────────────────┤
│ - floors: List<Floor> │
│ + parkVehicle(vehicle): Ticket │
│ + unparkVehicle(ticket): Fee   │
└───────┬────────┘
        │ has many
        ▼
┌────────────────┐        ┌──────────────────┐
│ Floor           │───────►│ ParkingSpot (abstract)│
├────────────────┤        ├──────────────────┤
│ - spots: List   │        │ - id, isOccupied  │
│ + findSpot(type)│        │ + park(vehicle)   │
└────────────────┘        │ + unpark()        │
                            └────────┬─────────┘
                     ┌────────────────┼────────────────┐
                     ▼                ▼                ▼
             ┌──────────────┐ ┌──────────────┐ ┌──────────────────┐
             │CompactSpot    │ │LargeSpot      │ │HandicappedSpot    │
             └──────────────┘ └──────────────┘ └──────────────────┘

┌────────────────┐        ┌──────────────────┐
│ Vehicle (abstract)│─────►│ Ticket             │
├────────────────┤        ├──────────────────┤
│ - licensePlate  │        │ - entryTime        │
│ - type          │        │ - spot: ParkingSpot│
└───────┬────────┘        │ + calculateFee()   │
        ▼                  └──────────────────┘
┌───────┬────────┬──────────┐
▼        ▼        ▼
Car   Motorcycle  Truck
```
```java
abstract class Vehicle {
    String licensePlate;
    VehicleType type;
}
class Car extends Vehicle {}
class Motorcycle extends Vehicle {}
class Truck extends Vehicle {}

enum VehicleType { MOTORCYCLE, CAR, TRUCK }
enum SpotType { COMPACT, LARGE, HANDICAPPED }

abstract class ParkingSpot {
    String id;
    boolean isOccupied;
    SpotType type;
    Vehicle currentVehicle;

    boolean canFitVehicle(Vehicle v) { /* size-compatibility rules */ return true; }

    void park(Vehicle v) {
        this.currentVehicle = v;
        this.isOccupied = true;
    }
    void unpark() {
        this.currentVehicle = null;
        this.isOccupied = false;
    }
}

class Floor {
    int floorNumber;
    List<ParkingSpot> spots;

    Optional<ParkingSpot> findAvailableSpot(Vehicle vehicle) {
        return spots.stream()
            .filter(s -> !s.isOccupied && s.canFitVehicle(vehicle))
            .findFirst();
    }
}

class Ticket {
    String ticketId;
    ParkingSpot spot;
    Instant entryTime;
    Instant exitTime;

    double calculateFee(FeeStrategy strategy) {
        return strategy.calculate(entryTime, exitTime, spot.type);
    }
}

interface FeeStrategy {
    double calculate(Instant entry, Instant exit, SpotType type);
}
class HourlyFeeStrategy implements FeeStrategy {
    public double calculate(Instant entry, Instant exit, SpotType type) {
        long hours = Duration.between(entry, exit).toHours() + 1; // round up
        double baseRate = switch (type) {
            case COMPACT -> 2.0;
            case LARGE -> 4.0;
            case HANDICAPPED -> 1.0;
        };
        return hours * baseRate;
    }
}

class ParkingLot {
    private static ParkingLot instance;
    private List<Floor> floors;
    private FeeStrategy feeStrategy;

    public static synchronized ParkingLot getInstance() {
        if (instance == null) instance = new ParkingLot();
        return instance;
    }

    public Ticket parkVehicle(Vehicle vehicle) {
        for (Floor floor : floors) {
            Optional<ParkingSpot> spot = floor.findAvailableSpot(vehicle);
            if (spot.isPresent()) {
                spot.get().park(vehicle);
                return new Ticket(UUID.randomUUID().toString(), spot.get(), Instant.now());
            }
        }
        throw new NoSpotAvailableException();
    }

    public double unparkVehicle(Ticket ticket) {
        ticket.exitTime = Instant.now();
        double fee = ticket.calculateFee(feeStrategy);
        ticket.spot.unpark();
        return fee;
    }
}
```
**Why this design:** Strategy pattern for fee calculation (easy to swap hourly/flat/dynamic pricing without touching core logic). Abstract `ParkingSpot`/`Vehicle` allow adding new types without modifying existing code (Open/Closed Principle). Singleton `ParkingLot` models a single physical facility instance — mention in interview that in a distributed/multi-location system you'd replace this with a DB-backed service instead of a true singleton.

---

## Practice 4: Design a Movie Ticket Booking System (like BookMyShow)

**Requirements:** Browse shows, select seats, hold seat temporarily during checkout, prevent double-booking under concurrency, payment.

### High-Level Architecture
```
┌────────┐   ┌───────────────┐   ┌─────────────────┐
│ Client  │──►│ API Gateway    │──►│ Booking Service   │
└────────┘   └───────────────┘   └────────┬──────────┘
                                            │
                       ┌────────────────────┼─────────────────┐
                       ▼                    ▼                 ▼
              ┌─────────────────┐  ┌────────────────┐ ┌────────────────┐
              │ Seat Lock Service│  │ Payment Service │ │ Notification   │
              │ (Redis, TTL lock)│  │                 │ │ Service (async)│
              └─────────────────┘  └────────────────┘ └────────────────┘
                       │
                       ▼
              ┌─────────────────┐
              │ DB (Shows, Seats,│
              │ Bookings) - SQL  │
              │ for ACID needs   │
              └─────────────────┘
```
**Key design decision & why:** Seat selection is the classic concurrency problem — two users must never book the same seat. Use a **short-lived distributed lock** (Redis `SETNX` with TTL, e.g., 5-10 min) when a user selects seats, so the seat is "held" during checkout; release on payment success/failure/timeout. Final booking confirmation still goes through a DB transaction with a unique constraint on (show_id, seat_id) as a safety net in case the lock layer fails.

### Class Diagram
```
┌───────────────┐      ┌───────────────┐      ┌───────────────┐
│  Movie         │      │  Show          │      │  Seat          │
├───────────────┤      ├───────────────┤      ├───────────────┤
│ - title        │◄─────│ - movie        │─────►│ - seatNumber   │
│ - duration     │ 1..* │ - theater      │ 1..* │ - type         │
└───────────────┘      │ - startTime    │      │ - status       │
                        └───────┬───────┘      └───────┬───────┘
                                │                       │
                                ▼                       ▼
                        ┌──────────────────────────────────┐
                        │  BookingService                    │
                        ├──────────────────────────────────┤
                        │ + holdSeats(showId, seatIds, user) │
                        │ + confirmBooking(holdId, payment)  │
                        │ + releaseSeats(holdId)              │
                        └──────────────┬───────────────────┘
                                       ▼
                        ┌──────────────────────────────────┐
                        │  Booking (entity)                  │
                        ├──────────────────────────────────┤
                        │ - bookingId, userId, showId         │
                        │ - seats: List<Seat>, status          │
                        └──────────────────────────────────┘
```
```java
enum SeatStatus { AVAILABLE, LOCKED, BOOKED }

class Seat {
    String seatId;
    SeatStatus status;
    SeatType type;
}

class SeatLockService {
    private final RedisClient redis;
    private static final int LOCK_TTL_SECONDS = 600;

    boolean tryLockSeats(String showId, List<String> seatIds, String userId) {
        // Use a Redis transaction/Lua script to atomically check-and-set
        // all seat keys; if ANY seat is already locked, abort and unlock none.
        for (String seatId : seatIds) {
            String key = "lock:" + showId + ":" + seatId;
            boolean acquired = redis.setIfAbsent(key, userId, LOCK_TTL_SECONDS);
            if (!acquired) {
                releaseSeats(showId, seatIds); // rollback partial locks
                return false;
            }
        }
        return true;
    }

    void releaseSeats(String showId, List<String> seatIds) {
        for (String seatId : seatIds) redis.delete("lock:" + showId + ":" + seatId);
    }
}

class BookingService {
    private final SeatLockService lockService;
    private final BookingRepository bookingRepository;
    private final PaymentService paymentService;

    public String holdSeats(String showId, List<String> seatIds, String userId) {
        boolean locked = lockService.tryLockSeats(showId, seatIds, userId);
        if (!locked) throw new SeatsUnavailableException();
        return bookingRepository.createPendingBooking(showId, seatIds, userId);
    }

    public Booking confirmBooking(String bookingId, PaymentDetails payment) {
        boolean paid = paymentService.charge(payment);
        if (!paid) {
            bookingRepository.cancel(bookingId);
            throw new PaymentFailedException();
        }
        // DB update guarded by unique constraint on (show_id, seat_id) as final safety net
        return bookingRepository.confirm(bookingId);
    }
}
```

---

## Practice mock-interview format (use for self-practice)
For any new problem, walk through in this order out loud, timed to ~35-40 min total:
1. **Clarify requirements** (5 min) — functional + non-functional (scale, latency, consistency needs)
2. **Back-of-envelope estimates** (3 min) — QPS, storage, bandwidth
3. **High-level design** (10 min) — draw boxes, explain data flow
4. **Deep dive** (10-12 min) — pick 1-2 components interviewer cares about (usually the hardest part: concurrency, consistency, or a specific algorithm)
5. **Data model / class design** (5 min) — key entities and relationships
6. **Bottlenecks & tradeoffs** (5 min) — what breaks at 10x scale, and how you'd fix it

**Other strong practice problems to try on your own using this same structure:** Design Twitter/X news feed, Design WhatsApp (messaging + delivery guarantees), Design an e-commerce inventory system, Design a distributed job scheduler, Design Instagram (media storage + feed), Design a food delivery ETA/dispatch system.
