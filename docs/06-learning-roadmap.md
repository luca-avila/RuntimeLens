# 6 — Learning Roadmap (Post-MVP)

> **Goal:** Write down the distributed-systems evolution of RuntimeLens *before*
> building it — so the learning path is intentional, staged, and motivated.

RuntimeLens is, first and foremost, a **learning vehicle** for backend
architecture and distributed systems. The MVP ([05-mvp-scope.md](05-mvp-scope.md))
deliberately excludes queues, Redis, Go, and service decomposition as "premature
complexity." That exclusion is correct **for shipping a product** — but those
exact technologies are the **point** of this project as a learning exercise.

This document reconciles the two: the items the MVP labels *Postponed* /
*Premature complexity* are not rejected — they are a **structured learning
roadmap**, intentionally deferred until the core loop works.

---

## Guiding Principles

1. **Prove the core loop first.** Do not start any stage below until the MVP is
   complete (a real app sends events and they are inspectable). This directly
   guards against the *"scope becomes too broad"* and *"too much polish delays
   the MVP"* risks named in [01-problem-definition.md](01-problem-definition.md).
2. **One pain point per stage.** Each stage exists to solve a concrete problem
   *felt* in the previous stage. The aim is to understand **why** a piece of
   infrastructure exists, not to add it because real products have it.
3. **Build the simple version, feel the limit, then add the machinery.** This is
   the fastest route to genuine "under the hood" understanding.
4. **Keep it runnable locally.** Every stage should still come up with
   `docker compose up`. Multi-region/Kubernetes/cloud scale stays out of scope.

---

## Learning Objectives (from project intent)

Topics this project is meant to teach, mapped to where they appear below:

| Topic | Stage |
|-------|-------|
| Log ingestion, event modeling, querying | Stage 0 (MVP) |
| Event processing, queues, async workers, backpressure | Stage 1 |
| Redis, caching, rate limiting, idempotency | Stage 2 |
| Go, concurrency, agents/collectors, batching & retries | Stage 3 |
| Service decomposition, service boundaries | Stage 4 |
| API gateway, load balancing, horizontal scaling | Stage 5 |
| Metrics, tracing, observability internals | Stage 6 |

Cross-references to the reading list: queues/backpressure/idempotency,
replication, and partitioning map directly onto **DDIA**; the systems-level
concerns (sockets, concurrency, memory) connect to **CSAPP**.

---

## Stage 0 — MVP: The Monolith (baseline)

**Architecture:** FastAPI monolith + PostgreSQL, behind Docker Compose.
`POST /ingest` validates the API key and writes the event **synchronously** to
Postgres. The dashboard reads directly from Postgres.

**What you learn:** API design, API-key auth, structured event modeling, and
indexing/querying for search and filtering.

**The pain point that motivates Stage 1:** synchronous ingestion couples request
latency to database write latency. Under a burst of events, the ingest endpoint
slows down or drops requests, and a slow DB write can fail an otherwise valid
event. Ingestion should accept fast and persist independently.

---

## Stage 1 — Async Ingestion with a Queue

**Add:** a message queue (start with **Redis Streams** for simplicity, or
RabbitMQ for "real" broker semantics) between the ingest endpoint and a
**worker** that persists events.

```
POST /ingest → enqueue (fast ACK) → [queue] → worker → PostgreSQL
```

**What you learn:**

- Producer/consumer decoupling and why it raises ingest throughput.
- **Backpressure** — what happens when producers outrun consumers.
- **Delivery semantics** — at-least-once vs at-most-once, and why exactly-once is
  hard.
- **Idempotency** — using an event ID to make re-delivery safe.
- The **outbox pattern** (already an interest from PuntoEntrega) and durability
  vs throughput trade-offs.

**Pain point → Stage 2:** the worker and dashboard hammer Postgres for the same
hot data (recent events, overview counts), and there is still nothing stopping a
single API key from flooding ingestion.

---

## Stage 2 — Redis for Caching, Rate Limiting & Dedup

**Add:** Redis as a fast path alongside Postgres.

**What you learn:**

- **Read caching** for overview stats and recent events; cache invalidation and
  TTLs (the "two hard things" in practice).
- **Rate limiting** per API key (token-bucket / sliding window) — directly
  addresses the *"API keys and abuse prevention are underestimated"* risk.
- **Deduplication** of repeated event IDs at the edge.

**Pain point → Stage 3:** sending events still requires apps to call the HTTP API
directly. Real systems want a lightweight local agent that batches, buffers, and
retries — and that is the perfect excuse to pick up Go.

---

## Stage 3 — Go Collector / Agent

**Add:** a standalone **Go** component — a lightweight forwarder/agent (and/or
CLI) that tails or receives logs locally, batches them, and ships them to
`/ingest` with retries and backoff.

```
app logs → [Go collector: batch + buffer + retry] → POST /ingest
```

**What you learn:**

- **Go** fundamentals and idioms (the desired next language).
- **Concurrency** with goroutines and channels — batching, worker pools, graceful
  shutdown.
- Resilient client design: **batching**, local buffering, **exponential backoff**,
  and at-least-once shipping from the client side.
- Why a collector/agent exists in real observability stacks (Vector, Fluent Bit,
  the Datadog agent).

> This is the deliberate Go entry point. The UX docs intentionally keep Go hidden
> from *users* unless it improves their experience — but as a *learning goal* the
> collector is a first-class objective.

**Pain point → Stage 4:** the monolith now does too many jobs (ingest API, query
API, workers) in one deployable. Scaling or changing one concern means
redeploying everything.

---

## Stage 4 — Service Decomposition

**Add:** split the monolith into focused services:

- **Ingest service** — accepts and enqueues events (write-optimized).
- **Query/API service** — serves the dashboard (read-optimized).
- **Worker(s)** — consume the queue and persist events.

**What you learn:**

- Drawing **service boundaries** by read vs write and by failure domain.
- Inter-service communication and shared vs separate datastores.
- Independent deployment and scaling of each concern.
- The real costs of decomposition (operational overhead, consistency) — i.e.
  *why not to do this in the MVP*.

**Pain point → Stage 5:** there are now multiple service instances and no single
front door, no TLS termination point, and no way to spread ingest load.

---

## Stage 5 — API Gateway & Load Balancing

**Add:** a gateway / reverse proxy (Nginx — already used in PuntoEntrega — or
Traefik) in front of the services, load-balancing multiple **ingest** instances.

**What you learn:**

- Routing, TLS termination, and a single entry point.
- **Load balancing** strategies and **health checks**.
- **Horizontal scaling** of a stateless ingest tier and why statelessness matters.

**Pain point → Stage 6:** the platform is now several moving parts and you have no
visibility into *its own* latency, queue depth, or error rates — ironic for a
monitoring tool.

---

## Stage 6 — Observe the Observability Platform

**Add:** instrument RuntimeLens itself — metrics (Prometheus), traces
(OpenTelemetry), and a dashboard of its internals (queue depth, ingest latency,
worker throughput, error rates).

**What you learn:**

- Metrics vs logs vs traces, and the **RED/USE** method.
- **OpenTelemetry** instrumentation and a metrics pipeline.
- Closing the loop: using the very signals this project is about to debug itself.

---

## Roadmap at a Glance

```
Stage 0  Monolith + Postgres                 ← MVP (must work first)
   │     pain: sync ingest couples latency to DB
Stage 1  Queue + async worker                 ← event processing, backpressure
   │     pain: hot reads + no abuse protection
Stage 2  Redis cache + rate limit + dedup     ← caching, rate limiting
   │     pain: apps must call HTTP directly
Stage 3  Go collector / agent                 ← Go, concurrency, batching
   │     pain: monolith does too many jobs
Stage 4  Service decomposition                ← boundaries, deployability
   │     pain: no front door / no scaling
Stage 5  API gateway + load balancing         ← routing, horizontal scaling
   │     pain: no insight into the platform itself
Stage 6  Self-observability (metrics/traces)  ← OTel, metrics pipeline
```

---

## Explicitly Still Out of Scope

Even as learning targets, these stay out unless a concrete need appears — they add
operational complexity without proportional learning value for a
single-developer, locally-runnable project:

- Kubernetes and multi-region deployment
- Sharding/partitioning and dedicated search infrastructure (Elasticsearch, etc.)
- Cold storage / data warehousing
- Multi-tenant billing and quota systems
- Plugin marketplace, admin panels, team/org invites

Some of these (partitioning, replication) are worth **studying** via DDIA and
sketching on paper without necessarily building them here.

---

## Relationship to the Other Docs

- The **product** view of these items lives in
  [05-mvp-scope.md](05-mvp-scope.md) under *Excluded* / *Postponed*.
- This document is the **learning** view: the same items, re-framed as an
  intentional, staged curriculum rather than a feature backlog.
- When a stage ships, fold any user-facing parts back into the UX docs
  (e.g. a Go collector becomes a real option on the Integrations screen in
  [03-text-wireframes.md](03-text-wireframes.md)).
