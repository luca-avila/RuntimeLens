# RuntimeLens

A lightweight monitoring tool for backend applications. RuntimeLens lets
developers collect, inspect, and understand structured logs/events from their
systems — knowing **what** happened, **when**, **where**, and with **what
context** — without standing up a heavy observability stack.

> Send a structured event from your backend → see it, search it, and debug it in
> a clean dashboard.

## Project Intent

RuntimeLens is **dual-purpose**:

1. **Primary — a learning vehicle.** A hands-on way to learn backend
   architecture and distributed systems by building a real ingestion/query
   platform and evolving it stage by stage: queues, Redis, a Go collector,
   service decomposition, an API gateway, and load balancing.
2. **Secondary — a potential developer product.** A focused event/log inbox for
   solo developers and small teams who find Datadog/Grafana/Sentry too heavy for
   early-stage projects.

These goals are kept separate on purpose. The **MVP stays deliberately lean** (a
FastAPI monolith + PostgreSQL) so the core value loop is proven first. The
distributed-systems machinery is a **staged roadmap layered on afterward** — each
stage motivated by a real pain point felt in the previous one, which is the best
way to understand *why* that infrastructure exists.

## Documentation

The full UX and product process lives in [`docs/`](docs/):

| # | Stage | Document |
|---|-------|----------|
| 1 | Idea / Problem Definition | [docs/01-problem-definition.md](docs/01-problem-definition.md) |
| 2 | UX Thinking | [docs/02-ux-thinking.md](docs/02-ux-thinking.md) |
| 3 | Text Wireframes | [docs/03-text-wireframes.md](docs/03-text-wireframes.md) |
| 4 | User Flows | [docs/04-user-flows.md](docs/04-user-flows.md) |
| 5 | MVP Scope | [docs/05-mvp-scope.md](docs/05-mvp-scope.md) |
| 6 | Learning Roadmap (post-MVP) | [docs/06-learning-roadmap.md](docs/06-learning-roadmap.md) |

Visual wireframes: [docs/RuntimeLens-Wireframes.pdf](docs/RuntimeLens-Wireframes.pdf)

**Working in this repo (AI or human):** read
[AGENTS.md](AGENTS.md) first — the project follows an AI-first **TDD with
vertical slices** workflow and explicit-architecture rules. (`CLAUDE.md` just
points to it.)

## Status

Pre-implementation. UX research and MVP scope are defined; the staged learning
roadmap is being written down ahead of building.

## Stack (MVP)

- **Backend:** Python / FastAPI
- **Database:** PostgreSQL
- **Infrastructure:** Docker / Docker Compose
- **Frontend:** TypeScript / React / Next.js

Later stages introduce a message queue, Redis, and a Go-based collector — see the
[learning roadmap](docs/06-learning-roadmap.md).
