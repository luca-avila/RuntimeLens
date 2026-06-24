# RuntimeLens — UX & Product Documentation

RuntimeLens is a lightweight monitoring tool for backend applications. It helps
developers collect, inspect, and understand structured logs/events from their
systems without setting up a complex observability stack.

It is **dual-purpose**: primarily a learning vehicle for backend architecture and
distributed systems, and secondarily a potential developer product. The MVP stays
lean so the core loop ships; the distributed-systems learning (queues, Redis, Go,
service decomposition, gateway/load balancing) is a staged roadmap layered on
afterward — see [06-learning-roadmap.md](06-learning-roadmap.md).

This folder documents the full UX process that precedes implementation, from the
initial problem definition through to the defined MVP scope and the post-MVP
learning roadmap.

## Process Overview

The product was shaped through five stages, each building on the last:

| # | Stage | Document | Question it answers |
|---|-------|----------|---------------------|
| 1 | Idea / Problem Definition | [01-problem-definition.md](01-problem-definition.md) | What problem are we solving, and for whom? |
| 2 | UX Thinking | [02-ux-thinking.md](02-ux-thinking.md) | What should the experience feel like? |
| 3 | Text Wireframes | [03-text-wireframes.md](03-text-wireframes.md) | How are the main screens structured? |
| 4 | User Flows | [04-user-flows.md](04-user-flows.md) | What are the main journeys and states? |
| 5 | MVP Scope | [05-mvp-scope.md](05-mvp-scope.md) | What do we build first — and what do we defer? |
| 6 | Learning Roadmap | [06-learning-roadmap.md](06-learning-roadmap.md) | How does it evolve into a distributed system after the MVP? |

## Visual Wireframes

The structural text wireframes (stage 3) were translated into visual wireframes:

- [RuntimeLens-Wireframes.pdf](RuntimeLens-Wireframes.pdf)

## The Core Value Loop

Everything in this documentation supports one core loop the MVP must prove:

> A developer can send structured backend events and inspect them easily in a
> dashboard — understanding **what** happened, **when**, **where**, and with
> **what context**.
