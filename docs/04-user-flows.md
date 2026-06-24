# 4 — User Flows

> **Goal:** Define the main user journeys, decision points, and required states.

The product should make it easy for a developer to:

1. Create a monitored project.
2. Connect an app through an API key.
3. Send the first event.
4. Inspect events.
5. Debug errors or backend flows.

---

## Happy Path

### Flow 1: First project setup

1. User signs up / logs in.
2. User lands on empty dashboard.
3. User creates a project.
4. User generates an API key.
5. User chooses an integration method:
   - cURL
   - Python/FastAPI
   - TypeScript/Node
   - Go agent/collector (later)
6. User sends a test event.
7. Event appears in the dashboard.
8. User opens event detail and confirms metadata was received.

**Required screens/states**

- Sign up / login
- Empty dashboard
- Create project form
- Project overview
- API key creation
- Integration/setup screen
- Events list
- Event detail

**Primary actions**

- Create project
- Generate API key
- Copy snippet
- Send test event
- View event
- Inspect metadata

---

### Flow 2: Debug an error

1. User opens a project.
2. User goes to Events.
3. User filters by `error` or `critical`.
4. User searches by message, request ID, order ID, payment ID, webhook ID, or
   job ID.
5. User opens event detail.
6. User inspects metadata, stack trace, source, environment, and raw payload.
7. User copies useful debugging values.

**Required states**

- Events loading
- Events loaded
- Filtered results
- No matching results
- Event detail opened
- Metadata expanded/collapsed

**Primary actions**

- Filter events
- Search events
- Open event
- Copy ID/metadata
- Clear filters

---

### Flow 3: Manage API keys

1. User opens project settings / API keys.
2. User creates a new API key.
3. User copies it once.
4. User sees the key listed as active.
5. User can revoke the key later.

**Required states**

- No API keys
- Key generated
- Key copied
- Active key list
- Revoke confirmation
- Key revoked

**Primary actions**

- Create key
- Copy key
- Revoke key
- Cancel revoke

---

## Edge Cases

### Validation errors

- Project name is empty.
- Project name already exists.
- API key name is empty.
- Event payload is invalid.
- Required fields are missing.
- Metadata is too large.
- Invalid event level.

### Empty states

- No projects yet.
- No API keys yet.
- No events received yet.
- No events match current filters.
- Event exists but has no metadata.

### Timeouts / failures

- Event ingestion request times out.
- Dashboard fails to load events.
- API key creation fails.
- Search/filter request fails.
- Test event is sent but does not appear immediately.

### Permission issues

- User tries to access a project they do not own.
- API key is invalid.
- API key was revoked.
- API key belongs to another project.
- User session expired.

### Partial states

- Project exists but has no API key.
- API key exists but no events were received.
- Event received without optional metadata.
- Event received but stack trace is missing.
- Integration setup started but not completed.

### Cancellation flows

- User cancels project creation.
- User cancels API key creation.
- User cancels API key revocation.
- User exits onboarding before sending first event.
- User clears filters after a failed search.

---

## Decision Points

### After creating account

User chooses:

- Create first project
- Explore dashboard

Result: If no project exists, guide user back to project creation.

### After creating project

User chooses:

- Generate API key now
- Go to project overview

Result: If no API key exists, show setup reminder.

### After generating API key

User chooses integration:

- cURL
- Python/FastAPI
- TypeScript/Node
- Go agent/collector (later)

Result: Show matching setup instructions and copyable snippet.

### In events list

User chooses:

- Filter by level
- Search by text/ID
- Change date range
- Open event detail
- Clear filters

Result: Events list updates based on selected filters.

### In event detail

User chooses:

- Expand metadata
- View raw payload
- Copy values
- Return to events list

Result: User can inspect context without losing their place.

### When revoking API key

User chooses:

- Confirm revoke
- Cancel

Result:

- Confirm → key becomes inactive.
- Cancel → key remains active.

---

## Output

### Complete Flow Map

```
Sign up / Login
    ↓
Empty dashboard
    ↓
Create project
    ↓
Generate API key
    ↓
Choose integration
    ├── cURL
    ├── Python/FastAPI
    ├── TypeScript/Node
    └── Go agent/collector (later)
    ↓
Send test event
    ↓
Events list
    ↓
Event detail
    ↓
Debug / inspect / copy context
```

### Required States

- Loading
- Success
- Empty
- Validation error
- Permission error
- Network error
- Partial setup
- Confirmation required
- No results
- Revoked/inactive credential

### Interaction Logic

- If user has no projects, show project creation as the primary action.
- If project has no API key, show API key setup as the primary action.
- If API key exists but no events exist, show integration instructions.
- If events exist, show events list as the default project view.
- If filters return no results, show a clear empty state and allow reset.
- If an API key is revoked, reject future ingestion requests with a clear error.
- If the user opens an event, preserve filters when returning to the list.
