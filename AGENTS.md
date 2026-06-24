# Agent Rules — RuntimeLens

Rules for any AI agent (and human) working in this repo. Read this and the
[`docs/`](docs/) before writing code. These rules exist to keep the project
coherent with how it is meant to be built **and learned**.

> RuntimeLens is **primarily a learning vehicle** for backend architecture and
> distributed systems, secondarily a product. Optimize work for *learning value
> and technical clarity*, not just for shipping fast. See
> [docs/01-problem-definition.md](docs/01-problem-definition.md) and
> [docs/06-learning-roadmap.md](docs/06-learning-roadmap.md).

---

## 1. Working Approach: AI-first TDD with Vertical Slices

This is the core process. Follow it.

- **Test-first, always.** Write a failing test before the implementation
  (red → green → refactor). Do not write production code without a test that
  requires it. Do not write more code than the current failing test demands.
- **Vertical slices, not horizontal layers.** Deliver one thin end-to-end
  capability at a time (e.g. "ingest a single event and read it back"), through
  every layer it touches — API → service → DB → query — rather than building a
  whole layer in isolation. Each slice should be demonstrable and tested on its
  own.
- **Smallest slice that proves the next bit of value.** Prefer a narrow slice
  that works end-to-end over a wide feature that is half-wired.
- **Refactor on green only.** Clean up structure once tests pass; never refactor
  and change behavior in the same red step.
- **AI-first, human-in-the-loop.** Agents may scaffold, generate, and refactor
  freely, but surface non-obvious design decisions for review (see §6). The goal
  is that the human *understands* every architectural choice, not just that it
  works.

---

## 2. Architecture Rules

- **Explicit over magic.** Favor explicit wiring, clear data flow, and obvious
  boundaries. Avoid framework magic, hidden abstractions, and implicit globals.
  If a dependency is needed, inject it.
- **Well-defined boundaries.** Keep clear seams between HTTP layer, application/
  domain logic, and persistence. Domain logic must not import web-framework or
  ORM internals directly — pass plain data across boundaries.
- **Keep the MVP a lean monolith.** FastAPI + PostgreSQL + Docker Compose. Do
  **not** introduce queues, Redis, a Go service, service decomposition, a
  gateway, or load balancing during the MVP. Those are deliberately staged in
  [docs/06-learning-roadmap.md](docs/06-learning-roadmap.md) — pulling them
  forward defeats the learning design and violates the project's own
  scope-creep risk.
- **Design the seam before the stage.** When building MVP code that a later stage
  will change (e.g. the `ingest` write path that becomes async in Stage 1), keep
  it behind a small interface so the swap is local. Note the seam in a comment;
  don't build the future implementation.
- **Structured events are the core model.** Treat the event/log record as a
  first-class, well-typed structure (level, message, timestamp, source,
  environment, request id, metadata, raw payload). Validate at the edge.
- **No premature abstraction.** Don't generalize until there are at least two
  real call sites. Duplication is cheaper than the wrong abstraction.

---

## 3. Code Rules

- **Match the surrounding code.** Naming, structure, comment density, and idioms
  should look like the file they live in.
- **Type everything.** Python: type hints + Pydantic models at boundaries.
  TypeScript: no `any` without a written reason. Validate external input; trust
  internal types.
- **Fail loudly and clearly.** Return precise errors (validation, auth, not
  found, permission) — see the state lists in
  [docs/04-user-flows.md](docs/04-user-flows.md). No silent `except: pass`.
- **Security at the ingestion edge.** API-key auth on `POST /ingest`; never log
  secrets or full API keys; reject revoked keys; treat all event payloads as
  untrusted input. Abuse/rate-limit handling is a Stage 2 concern, but never
  weaken the auth check.
- **Comments explain *why*, not *what*.** Especially at boundaries and seams.
- **Small functions, clear names.** Prefer readable, composable units over
  clever one-liners.

---

## 4. Testing Rules

- **The test pyramid.** Many fast unit tests on domain logic; fewer integration
  tests across the API↔DB slice; minimal end-to-end. Each vertical slice ships
  with the tests that cover it.
- **Test behavior, not implementation.** Assert on observable outcomes and
  contracts, so refactors don't break tests.
- **Cover the states explicitly.** For each slice, test the relevant states from
  the flows doc: success, validation error, permission error, empty, not found.
- **Real Postgres in integration tests.** Use the Docker Compose / a disposable
  container, not an in-memory substitute, so behavior matches production.
- **A bug starts with a failing test** that reproduces it, then the fix.
- **Keep the suite fast and deterministic.** No reliance on wall-clock time,
  network, or test ordering.

---

## 5. Scope & Roadmap Discipline

- The MVP is "done" only per the boundary in
  [docs/05-mvp-scope.md](docs/05-mvp-scope.md). Don't add excluded features to
  "round it out."
- A roadmap stage is started only after the previous one works and its pain
  point is actually felt — not because the tech is interesting in the abstract.
- When a stage adds something user-facing (e.g. the Go collector as an
  integration option), update the relevant UX doc in the same change.

---

## 6. Learning-Mode Rules (because the primary goal is learning)

- **Explain the *why* of architectural choices.** When choosing between
  approaches (data model, boundary, concurrency, indexing, queue semantics),
  briefly state the trade-off and the reasoning — don't just pick silently.
- **Go beyond framework usage.** Where relevant, note what happens *under the
  hood* (e.g. how an index changes the query plan, what a transaction guarantees,
  why a write path is sync vs async). This is the point of the project.
- **Don't hide the hard parts.** Surface concurrency, consistency, idempotency,
  and failure-mode decisions for discussion rather than burying them.
- **Prefer teaching a pattern once over applying it silently many times.** If a
  new pattern is introduced, make its first use a clear, commented example.
- **Connect to the reading list when natural** (DDIA for
  queues/replication/partitioning; CSAPP for systems-level concerns).

---

## 7. Conventions

- **Local-first.** Everything must run with `docker compose up`. No cloud
  dependency to develop or test.
- **Commits:** small, behavior-focused, ideally one vertical slice or one
  refactor each. Don't mix a refactor with a behavior change.
- **Docs are source of truth for intent.** If code intentionally diverges from
  the docs, update the docs in the same change.
