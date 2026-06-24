# 5 — MVP Scope

> **Goal:** Reduce complexity and maximize learning speed.

The MVP should prove one core value:

> A developer can send structured backend events and inspect them easily in a
> dashboard.

The goal is **not** to build a full observability platform yet.

---

## Included

### Core functionality

- User registration/login
- Project creation
- API key generation/revocation
- `POST /ingest` endpoint
- Structured event/log storage
- Events list
- Event detail page
- Search and basic filters
- Project overview
- Integration/setup screen
- Docker Compose setup
- Basic README

### Must-have flows

- Create account
- Create project
- Generate API key
- Send first test event
- View received events
- Filter/search events
- Open event detail
- Inspect metadata
- Revoke API key

### Initial integrations

- cURL snippet
- Python/FastAPI snippet
- TypeScript/Node snippet

> Go can be added later only if it clearly fits a specific component, such as a
> collector, CLI, agent, or log forwarder.

---

## Excluded

### Nice-to-have features

- Advanced charts
- Custom dashboards
- Team management
- Billing
- Public status pages
- Slack/Discord/email alerts
- AI summaries
- Full-text search engine
- Complex role-based permissions
- Multi-region deployment
- Mobile UI optimization

### Premature complexity

- Kubernetes
- OpenTelemetry compatibility
- Distributed tracing
- Metrics pipeline
- Log aggregation at high scale
- Event streaming architecture
- Dedicated data warehouse
- Microservices
- Complex worker system
- Horizontal scaling

---

## Postponed

> **Note for this project:** many items below — queues, Redis, a Go
> collector, service decomposition, an API gateway, load balancing — are not just
> a product backlog. They are an intentional, staged **learning roadmap**, since
> RuntimeLens is primarily a vehicle for learning backend/distributed systems.
> The "premature complexity" framing still holds: they are deliberately deferred
> until the core loop works, then added one pain point at a time. See
> [06-learning-roadmap.md](06-learning-roadmap.md).

### Future ideas

- Go-based agent/collector
- CLI tool
- SDK packages
- Alert rules
- Error grouping
- Incident reports
- AI-assisted debugging
- Webhook replay/debugging module
- Retention policies
- Export events as CSV/JSON
- OpenTelemetry ingestion
- Grafana integration

### Scaling concerns

- High-volume ingestion
- Sharding/partitioning
- Queue-based buffering
- Cold storage
- Multi-tenant billing limits
- Advanced rate limiting
- Dedicated search infrastructure

### Secondary systems

- Admin panel
- Audit logs
- Organization/team invites
- Usage analytics
- Notification system
- Plugin marketplace

---

## Clear MVP Boundary

The MVP is complete when:

- A user can create a project.
- A user can generate an API key.
- An external app can send structured events.
- Events are persisted in PostgreSQL.
- The user can search, filter, and inspect events.
- The setup can run locally with Docker Compose.
- A demo app successfully sends real events.

> Anything beyond this is postponed unless it directly helps validate the core
> loop.
