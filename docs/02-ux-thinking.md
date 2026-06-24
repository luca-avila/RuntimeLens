# 2 — UX Thinking

> **Goal:** Design the experience before designing the system.

The product should help a developer go from:

> "Something happened in my backend"

to:

> "I can see what happened, when, where, and with what context."

The UX priority is **fast setup, clear event inspection, and easy debugging**.

---

## User / Problem Research

Initial users:

- Solo developers
- Small backend teams
- Developers running MVPs
- Developers deploying to VPS, Docker, or simple cloud setups
- Developers working with webhooks, payments, workers, external APIs, or backend
  services

Main pain points:

- Logs are scattered across terminal, Docker, VPS, and cloud dashboards.
- Debugging production-like issues is slow.
- Existing tools can be too complex or expensive.
- Webhook/payment/job failures are hard to inspect after they happen.
- Setup friction kills adoption.

---

## Information Architecture

Main objects:

- **Workspace** — owner/team container.
- **Project** — monitored application.
- **API Key** — credential used by an app to send events.
- **Event/Log** — structured record of something that happened.
- **Event Detail** — full context for debugging.
- **Integration** — way to send events into the platform.

Basic structure:

```
Workspace
└── Projects
    └── Project
        ├── Overview
        ├── Events
        ├── Event Detail
        ├── API Keys
        ├── Integrations
        └── Settings
```

---

## Developer Experience

The setup experience is part of the UX. The product should make integration feel
simple:

- Start with cURL.
- Add Python/FastAPI snippet.
- Add Node/TypeScript snippet.
- Add Go only when it clearly fits a specific component, such as:
  - log collector
  - host agent
  - CLI
  - lightweight forwarder
  - concurrent worker
  - infrastructure tooling

> Go should not be exposed as extra complexity unless it improves the user
> experience.

---

## Visual UI Exploration

The interface should feel:

- Technical
- Clear
- Fast
- Calm
- Developer-oriented

Initial visual direction:

- Simple dashboard layout
- Tables for events
- Badges for severity
- JSON/code blocks for metadata
- Copy buttons for IDs and snippets
- Minimal charts in the MVP

> Avoid making it feel like a generic admin panel or a clone of
> Datadog/Grafana/Sentry.

---

## Design System Thinking

Core reusable components:

- App shell
- Sidebar
- Page header
- Stat card
- Events table
- Filter bar
- Search input
- Status/severity badge
- JSON viewer
- Code snippet block
- Copy button
- Empty state
- Confirm dialog

Important empty states:

- No projects → create first project
- No API key → generate one
- No events → send test event
- No integration → choose setup method
- No results → adjust filters

---

## UX Success Criteria

The UX is successful if:

- A new user can send the first event quickly.
- A returning user can find recent errors easily.
- Event metadata is easy to inspect.
- API key setup is clear.
- Integration instructions are simple.
- The product feels useful before advanced features exist.
