# 1 — Idea / Problem Definition

> **Goal:** Understand the problem before thinking about implementation.

RuntimeLens is a lightweight monitoring tool for backend applications. It helps
developers collect, inspect, and understand structured logs/events from their
systems without setting up a complex observability stack.

> **Project intent (resolved):** RuntimeLens is *primarily a learning vehicle*
> for backend architecture and distributed systems, and *secondarily a potential
> developer product*. This resolves the "portfolio vs open-source vs SaaS"
> unknown below: it starts as a learning/portfolio project with a clear path to
> becoming a product. The implication is that some technologies the MVP defers as
> "premature complexity" (queues, Redis, Go, service decomposition) are
> intentional learning goals — see [06-learning-roadmap.md](06-learning-roadmap.md).

---

## Context

Small backend apps often generate important signals: errors, webhooks, payment
events, background job failures, API failures, retries, and custom business
events.

The problem is that this information is usually scattered across terminal
output, Docker logs, VPS files, cloud dashboards, or third-party tools.

---

## Who Has the Problem

Initial users:

- Solo developers
- Small backend teams
- Developers running MVPs
- Developers deploying to VPS / Docker / simple cloud setups
- Developers working with webhooks, payments, workers, and external APIs

---

## Why It Matters

When backend behavior is hard to inspect, debugging becomes slow and uncertain.

This is especially painful when issues happen outside local development, during
flows like payments, webhook processing, retries, background jobs, or
third-party integrations.

---

## Current Alternatives

Developers currently use:

- `print()` / console logs
- Docker logs
- VPS log files
- Cloud provider logs
- Sentry
- Datadog
- Grafana / Loki / Prometheus
- ELK
- Manual DB inspection

These tools can work, but they may be too complex, expensive, fragmented, or
overkill for small projects.

---

## Problem Statement

> Solo developers and small backend teams need a simple way to send, search, and
> inspect structured logs/events from real applications.

Existing observability tools are powerful, but often too heavy for early-stage
projects. The opportunity is to build a focused **event/log inbox** that helps
developers understand what happened in their backend — when, where, and with
what context.

---

## Success Criteria

MVP success means:

- User can create an account.
- User can create a project.
- User can generate an API key.
- External apps can send events to `POST /ingest`.
- Events are stored in PostgreSQL.
- User can view, search, and filter events.
- User can inspect event metadata.
- The app runs locally with Docker Compose.
- At least one real/demo app sends events successfully.

---

## Initial Assumptions

- Structured events are more useful than plain text logs.
- API key ingestion is the right auth model for external apps.
- PostgreSQL is enough for the MVP.
- Search/filtering matters more than charts at the beginning.
- The first value moment is seeing a real event appear in the dashboard.
- The project can start as a portfolio/learning tool and later evolve into a
  product.

---

## Risks / Unknowns

### Main risks

- Scope becomes too broad.
- The product imitates Datadog/Grafana/Sentry too early.
- Ingestion volume becomes hard to manage.
- API keys and abuse prevention are underestimated.
- The dashboard becomes generic instead of debugging-focused.
- Too much polish delays the MVP.

### Unknowns to validate

- Should the first SDK target FastAPI, Node, or Go?
- Should the MVP focus on logs, errors, webhooks, or generic events?
- What metadata fields are required?
- What is the simplest useful alerting mechanism?
- Is this mainly a portfolio project, open-source tool, or future SaaS?
