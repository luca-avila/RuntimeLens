# 3 — Text Wireframes

> **Goal:** Define the structure of the main screens before creating visual
> wireframes in Figma.

These wireframes describe layout, hierarchy, primary actions, and required states
without focusing on visual design yet. The resulting visual wireframes live in
[RuntimeLens-Wireframes.pdf](RuntimeLens-Wireframes.pdf).

---

## Screen 1: Projects List

**Purpose:** Let the user see and open their monitored applications.

**Main content**

- Page title: `Projects`
- Button: `Create project`
- List/table of projects:
  - Project name
  - Latest event time
  - Errors in last 24h
  - Status/health indicator

**Primary actions**

- Create project
- Open project

**States**

- Loading projects
- No projects yet
- Projects loaded
- Error loading projects

---

## Screen 2: Project Overview

**Purpose:** Give a quick summary of recent activity for one project.

**Main content**

- Project name
- Events in last 24h
- Errors in last 24h
- Warnings in last 24h
- Latest events
- Setup status:
  - API key exists
  - Last event received

**Primary actions**

- View all events
- Go to setup/integrations
- Manage API keys

**States**

- No API key
- API key exists but no events
- Events received
- Error loading overview

---

## Screen 3: Events List

**Purpose:** Let the user search, filter, and browse backend events.

**Main content**

- Search input
- Filters:
  - Level
  - Date range
  - Source
  - Environment
- Events table:
  - Timestamp
  - Level
  - Message
  - Source
  - Environment
  - Metadata preview

**Primary actions**

- Search events
- Filter events
- Open event detail
- Clear filters

**States**

- Loading events
- No events yet
- No matching results
- Events loaded
- Error loading events

---

## Screen 4: Event Detail

**Purpose:** Help the user understand a specific backend event.

**Main content**

- Event message
- Level
- Timestamp
- Source
- Environment
- Request ID, if available
- Metadata
- Stack trace, if available
- Raw payload

**Primary actions**

- Copy event ID
- Copy metadata values
- Expand/collapse metadata
- Return to events list

**States**

- Loading event
- Event loaded
- Event has no metadata
- Event not found
- Permission error

---

## Screen 5: API Keys

**Purpose:** Let the user manage ingestion credentials for a project.

**Main content**

- Existing API keys
- Key name
- Created date
- Last used date
- Status
- Revoke action

**Primary actions**

- Create API key
- Copy new key
- Revoke key
- Cancel revoke

**States**

- No API keys
- Key created
- Key copied
- Active keys
- Revoked key
- Error creating key

---

## Screen 6: Integrations / Setup

**Purpose:** Help the user send the first event.

**Main content**

- Setup steps:
  1. Create project
  2. Generate API key
  3. Choose integration
  4. Send test event
  5. See event in dashboard
- Integration options:
  - cURL
  - Python/FastAPI
  - TypeScript/Node
  - Go agent/collector (later)
- Copyable code snippet

**Primary actions**

- Choose integration
- Copy snippet
- Send/view test event
- Go to events list

**States**

- No API key
- API key ready
- Waiting for first event
- First event received
- Test event failed
