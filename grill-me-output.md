# RuntimeLens — "Grill Me" Session Output

> **Purpose:** Challenge assumptions before implementation. Each entry records the
> question (the assumption being challenged), the decision made, and any tension
> or follow-up it created. This is a living doc — the session was paused partway;
> see **[Resume Here](#resume-here)** to continue.
>
> **Status:** Paused after 11 answered decisions. 1 question pending, more queued.
> **Date:** 2026-06-24.

---

## Context

RuntimeLens is a lightweight backend monitoring tool — a structured **event/log
inbox**. Core loop the MVP must prove: *a developer can send structured backend
events and inspect them (what / when / where / context) in a dashboard.*

- **Primary goal:** learning vehicle for backend architecture & distributed
  systems. Secondary: a potential product.
- **MVP stack (Stage 0):** FastAPI + PostgreSQL + Docker Compose, **synchronous**
  `POST /ingest`, dashboard reads directly from Postgres.
- **Process rules (AGENTS.md):** test-first TDD, vertical slices, explicit
  boundaries, lean monolith (NO queues/Redis/Go/decomposition in MVP — those are
  the staged roadmap), docs are source of truth (update docs when code diverges).
- **Post-MVP roadmap:** Stage 1 queue/async worker → Stage 2 Redis
  cache/rate-limit/dedup → Stage 3 Go collector → Stage 4 service decomposition →
  Stage 5 gateway/LB → Stage 6 self-observability.

---

## Decisions Log

### 1. Tenancy model — **Full workspace + members**
**Challenge:** IA defines Workspace as an "owner/team container" (Workspace →
Projects), but MVP scope explicitly **excludes** Team management and complex RBAC.
**Decision:** Build `users`, `workspaces`, `workspace_members (ws_id, user_id,
role)`, `projects (ws_id, …)` now.
**⚠️ Tension to revisit:** This contradicts `docs/05-mvp-scope.md` (Team
management + role-based permissions are listed under *Excluded*) and front-loads
complexity the roadmap defers. If kept, **update the MVP scope doc** to legitimize
it, and decide the minimum role set. Otherwise reconsider the lighter "implicit
1:1 personal workspace" option (workspace entity, membership/roles deferred).

### 2. Event contract (required fields) — **Only `level` + `message` required**
**Challenge:** Field list existed but required-vs-optional was never defined.
**Decision:**
- **Required:** `level`, `message`
- **Server-set:** `id`, `received_at`
- **Optional:** `source`, `environment`, `request_id`, `metadata`, `timestamp`
Rationale: lowest friction for the "first event" UX.

### 3. Level enum — **5 standard levels**
**Challenge:** "Invalid event level" implied a fixed enum that was never listed.
**Decision:** `debug | info | warning | error | critical`. Reject anything else
with `422`. Matches Python logging + the docs' "filter by error or critical".

### 4. Time authority — **Client time, server fallback**
**Challenge:** Two timestamps (client `timestamp` vs server `received_at`) but
sorting, 24h counts, and date filters need ONE authoritative time.
**Decision:** `effective = timestamp ?? received_at`; order & filter by
`effective`. Both columns stored.
**⚠️ Risk to mitigate:** client clock skew / wrong TZ / batched-late sends can
corrupt ordering and the "last 24h" windows. **Recommended guard (queued as an
open question):** reject or clamp timestamps that are too far in the future
(and optionally too far in the past); always keep `received_at` as the tamper-
proof anchor.

### 5. Search scope (MVP) — **`message` ILIKE + `request_id` exact only**
**Challenge:** Flow 2 promises search by order/payment/webhook/job IDs, but those
live in arbitrary `metadata` JSON, and MVP excludes a full-text search engine.
**Decision:** `WHERE message ILIKE '%q%' OR request_id = 'q'`. Metadata IDs **not
searchable** in MVP.
**⚠️ Doc fix needed:** `docs/04-user-flows.md` Flow 2 over-promises — update it to
say metadata-ID search is deferred (candidate for the Stage 2 JSONB-GIN or
full-text work).

### 6. Payload limits — **Hard caps, reject over**
**Challenge:** "Metadata is too large" was a listed error with no defined limit;
unbounded payloads are a DoS/DB-bloat vector.
**Decision:** `body ≤ 64KB`, `message ≤ 8KB`, `metadata ≤ 16KB`. Exceed →
`413/422` with a clear error. (Numbers are a starting point; tune later.)

### 7. API key storage — **Re-viewable + encrypted at rest**
**Challenge:** A hash is one-way, so "re-viewable" and "store only a hash" are
mutually exclusive; current docs assume **show-once**.
**Decision:** Keys can be revealed again in the UI; stored **encrypted at rest**
with an app-held secret. UI "reveal" → decrypt + show.
**⚠️ Consequences to handle:**
- Contradicts `docs/04-user-flows.md` Flow 3 ("User copies it once") and Screen 5
  states in `docs/03-text-wireframes.md` → **update those docs**.
- Introduces an **encryption-key management** dependency (where the app secret
  lives, rotation). Design behind a small interface (seam).
- Reversible: a leak of DB **and** app key exposes all keys — note this in code.

### 8. Pagination — **Keyset / cursor**
**Challenge:** Core read surface grows unbounded; pagination was unspecified.
**Decision:** Cursor on `(effective_time, id)`:
`WHERE (eff, id) < (cur_t, cur_id) ORDER BY eff DESC, id DESC LIMIT 50`, backed by
an index on `(eff DESC, id DESC)`. Stable under live inserts; teaches keyset
pagination.

### 9. Ingest shape — **Single event per request**
**Challenge:** Stage 3 collector batches; should MVP accept arrays now?
**Decision:** One event per request: `POST /ingest { level, message, … } → 201
{ id }`. Batch endpoint deferred to Stage 3 (avoids partial-failure semantics
now). Keeps the cleanest seam.

### 10. Event ID / idempotency — **Server id + optional client key (unenforced)**
**Challenge:** Timeout-retries create duplicates; dedup is Stage 2 but the ID
strategy is a seam decided now.
**Decision:** Server always assigns `id = uuid`. Accept an **optional**
`client_event_id` (stored, **not** enforced in MVP). Stage 2 adds
`UNIQUE(project_id, client_event_id)` + dedup with no contract change.

### 11. Frontend stack — **Next.js + TypeScript + Tailwind**
**Challenge:** Rich dashboard described but no framework chosen; backend is the
primary learning goal.
**Decision:** Next.js + TS + Tailwind, consuming the FastAPI JSON API.
**Constraint (to protect the learning goal + Stage 4 decomposition):** Next.js is
**presentation-only** — all domain logic and DB access go through the FastAPI JSON
API, NOT Next.js server routes / server components hitting Postgres.

### 12. Session auth (dashboard) — **HttpOnly cookie session**
**Challenge:** Separate Next.js origin needs a session scheme (distinct from the
API-key auth on `/ingest`); "User session expired" was listed but unspecified.
**Decision:** FastAPI sets a signed **HttpOnly, Secure, SameSite=Lax** cookie.
Requires **CSRF token on mutations** and **CORS-with-credentials** config.
XSS-safe token storage.

---

## Resume Here

### ⏸ Pending question (was being asked when paused)

**Abuse guard in MVP?** Rate limiting is deliberately a Stage 2 topic, yet "API
keys and abuse prevention are underestimated" is a named MVP risk, and Stage 0
ingest is synchronous (a flood stalls the DB directly). Options on the table:
1. **Strictly defer (docs win)** — auth + size caps only; preserve the Stage 2
   pain point exactly. *(Leading candidate — aligns with roadmap discipline.)*
2. **Crude per-key cap** — in-process N/s → 429; muddies Stage 2 motivation.
3. **Defer + document the risk** — no limiter, but mark the exposure explicitly so
   it's a conscious gap, not an oversight.

### 🔜 Queued questions (not yet asked)

1. **Future/past timestamp guard** (follow-up to Decision #4) — reject or clamp
   client timestamps outside a sane window so skewed clocks can't poison the
   "recent / last 24h" views.
2. **Retention / data growth** — events grow unbounded; any MVP retention or is
   it explicitly out of scope? (Roadmap lists retention policies as Postponed.)
3. **Demo app** — success criteria require "at least one real/demo app sends
   events". What is it (which language/scenario), and is it in-repo?
4. **Permission enforcement model** — now that Workspace+members+roles is chosen
   (Decision #1), what exactly is checked on each request (project ownership,
   role gates), and which roles exist?
5. **`environment` field** — free-text vs enum (e.g. production/staging/dev)? It's
   a filter, so consistency matters.
6. **`source` field** — what does it mean (service name? logger name?), and is it
   validated/normalized?
7. **Sync→async ingest seam** — confirm the interface shape so Stage 1 swaps the
   write path locally (AGENTS.md §2 already mandates this).
8. **Project name validation** — "name already exists": unique per workspace?
   case-insensitive? empty-name handling.
9. **API key scoping** — confirm keys are per-project; "API key belongs to
   another project" must be rejected. Cross-project behavior.
10. **Stack trace & raw payload** — is `stack_trace` a first-class field or part
    of metadata? Is `raw payload` the verbatim request body stored alongside the
    parsed fields?

### 📌 Cross-cutting: docs that must be updated (per AGENTS.md §7)

Decisions so far intentionally diverge from the current docs. Before/while
implementing, reconcile:
- `docs/05-mvp-scope.md` — workspace/members/roles now in MVP (Decision #1).
- `docs/04-user-flows.md` — Flow 2 metadata-ID search deferred (#5); Flow 3
  API-key "copies it once" → re-viewable (#7).
- `docs/03-text-wireframes.md` — Screen 5 API-key states reflect re-viewable
  (#7).
