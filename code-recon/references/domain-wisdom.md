# Compressed Wisdom Seeds for Scalable Systems

These are not rules. They are seeds. Plant them in the soil of your problem and they will grow into architecture.

---

## Seed 1: "Each Organ Has One Job"

Databases store truth and run maintenance. They do not dispatch HTTP requests. Schedulers distribute work. They are not databases. Caches serve static content. They are not origins. Projections serve public queries. They are not raw tables. Buffers absorb writes. They are not hot rows.

**Violation:** Database becomes job queue → hits background-worker ceiling → checks drift late.  
**Honored:** Each system does its one thing. The whole scales.

---

## Seed 2: "Don't Build a House on Sand"

Scalability is not an afterthought. It's the foundation. Test at 1x, 10x, 100x load before shipping. When something breaks under pressure, you've found where the sand is. Fix the sand, not the house.

---

## Seed 3: "The River Doesn't Fight the Rocks"

Don't fight lock contention. Eliminate it by changing the write pattern. Append-only buffers have no contention. Batching amortizes cost.

**Violation:** Multiple processes fight over the same row. ~200 writes/sec ceiling.  
**Honored:** Same hardware, 25x throughput.

---

## Seed 4: "Projection Is Cheaper Than Computation"

Pre-compute current state. Store it in a tiny, indexed table. Public pages read the map, not the terrain. One indexed lookup scales better than aggregating a million rows.

---

## Seed 5: "The Edge Is Closer Than the Origin"

Cache public content at the edge (Cloudflare, CDN). A million requests become ~60 origin hits. The origin only handles cache misses.

**Violation:** Every request hits your origin. Origin becomes the bottleneck.  
**Honored:** 99% of traffic served from edge. Origin handles 1%.

---

## Seed 6: "The Right Tool for the Right Job"

pg_cron is great for SQL maintenance. Terrible for HTTP fan-out. Schedulers are designed for distributed work. Databases aren't. When you hit a tool's limit, you've violated this seed.

---

## Seed 7: "Know What Breaks If You Delete This"

Before you remove a brick from a wall, know which wall it's holding up. Ask: What files depend on this? What data flows through this? What happens if this fails halfway? What's the rollback path? What's the actual blast radius?

---

## Seed 8: "The Smallest Cut Heals Fastest"

Don't rewrite half the app because one query is slow. Don't refactor for cleanliness while fixing production. Don't touch unrelated files. The smallest safe change always wins.

---

## Seed 9: "Measure Before You Optimize"

Load test until something breaks. Measure throughput, latency, errors, resource usage. Document what breaks and at what load. Fix the break. Measure again. Extract the invariant. Codify it.

---

## Seed 10: "Scalability Is Not a Feature. It's a Foundation."

Design for 10x, 100x load from the start. Prototype for features. Production for scale. When something breaks under load, you haven't violated this seed. You've discovered where the foundation is weak.

---

## Seed 11: "Load Testing Is Discovery, Not Validation"

You don't load test to prove something works. You load test to find where it breaks. The break is the discovery. Each failure teaches you something. Extract the lesson. Codify it.

---

## Seed 12: "Contention Is an Architecture Problem, Not a Database Problem"

Multiple processes fighting over the same row = lock contention. This is not a Postgres problem. It's an architecture problem. Append-only buffers = many doors. Hot-row updates = one door.

---

## Seed 13: "The Database Should Store Truth, Not Dispatch Work"

Databases store truth. They run maintenance (cheap SQL jobs). Schedulers distribute work (HTTP fan-out, event processing). Never ask the database to be a courier.

**Violation:** pg_cron + pg_net becomes an HTTP dispatcher → background-worker ceiling → checks drift late.

---

## Seed 14: "Public Traffic Reads From Projections, Not Raw Data"

Pre-compute current state. Store in tiny, indexed tables. Public pages read projections (fast, indexed). Analytics can still scan raw data (slow, but not on public path).

---

## Seed 15: "Append-Only Buffers Eliminate Contention"

Writes go to append-only buffer (no contention). Background flusher batches updates (amortizes cost). Current-state projections always available (fast reads). Same hardware, 25x throughput.

---

## Seed 16: "Separate Connection Pools for Different Workloads"

Don't make background workers wait in line behind public traffic. Public traffic uses one connection pool. Background workers use another. Each can scale independently.

---

## Seed 17: "Idempotent Jobs Everywhere"

A job should be safe to run twice. Jobs can be retried without side effects. Partial failures are recoverable.

---

## Seed 18: "Graceful Degradation Over Hard Failure"

When the road is crowded, traffic moves slowly — it doesn't stop. Requests get slower response times, not 500 errors. Users see slowness, not broken.

---

## Seed 19: "Jittered Scheduling Over Synchronized Bursts"

Don't schedule all jobs at the same time. Add jitter so jobs spread out. System load is smooth, not spiky.

---

## Seed 20: "The Preference Hierarchy Is Not Arbitrary"

Prefer:
- Scheduler → queue → workers → reducers (over direct synchronous processing)
- Append-only buffers + rollups (over constant hot-row updates)
- Current-state projections (over live aggregation queries)
- External scheduler + controlled fan-out (over database as dispatcher)
- Edge caching (over origin scaling)
- Partitioned high-write tables (over giant single tables)
- Jittered scheduling (over synchronized bursts)
- Idempotent jobs (over exactly-once guarantees)
- Separate connection pools (over single shared pool)
- Graceful degradation (over hard failures)

---

## The Meta-Seed

> "Architecture matters more than platform. Scalability is not an afterthought. The smallest safe change always wins. Always know your blast radius. Measure everything. Optimize only what you've measured. Move fast, but move carefully. The system will tell you when you're wrong. Listen to it."
