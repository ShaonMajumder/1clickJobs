# Minimal Job Tracker System Design

## Architecture

```text
Single HTML App
      ↓
IndexedDB
      ├── jobs
      └── sources
      ↓
JSON Export / Import
```

No backend, login, scraping, or AI.

---

# Core Flow

```text
User pastes job link
      ↓
App saves immediately
      ↓
Default status = Open
      ↓
App extracts source domain
      ↓
If source does not exist, create it
```

---

# `jobs` Store

```js
{
  id: "uuid",
  jobUrl: "https://linkedin.com/jobs/view/123",
  sourceDomain: "linkedin.com",
  status: "Open",
  createdAt: "...",
  updatedAt: "..."
}
```

Indexes:

```text
jobUrl
sourceDomain
status
createdAt
updatedAt
```

---

# Job Statuses

```text
Open
Applied
Interview
Offer
Rejected
Ghosted
Closed
```

## Meaning

| Status | Meaning |
|---|---|
| Open | Saved/tracked but not applied yet |
| Applied | Application submitted |
| Interview | Interview process started |
| Offer | Received offer |
| Rejected | Explicit rejection |
| Ghosted | No response for a long time |
| Closed | Job opening no longer available |

---

# `sources` Store

The `sources` table must store both the normalized domain and the original pasted source URL.

```js
{
  id: "uuid",
  domain: "linkedin.com",
  sourceUrl: "https://www.linkedin.com/jobs",
  displayName: "LinkedIn",
  isPortal: true,
  createdAt: "...",
  updatedAt: "..."
}
```

## Fields

| Field | Purpose |
|---|---|
| domain | extracted normalized domain used for grouping/filtering |
| sourceUrl | original pasted source/portal URL |
| displayName | editable display name shown in UI |
| isPortal | whether this source is a job portal/job board |
| createdAt | source creation timestamp |
| updatedAt | source update timestamp |

Indexes:

```text
domain
sourceUrl
displayName
isPortal
```

---

# Source Rules

## Auto-created source from job link

When a user pastes a job link:

```text
https://www.linkedin.com/jobs/view/123
```

App extracts the domain:

```text
linkedin.com
```

Then:

```text
If linkedin.com does not exist in sources:
  create source
  domain = "linkedin.com"
  sourceUrl = "https://www.linkedin.com/jobs/view/123"
  displayName = "linkedin.com"
  isPortal = true
```

## Manual source creation

Manual source creation is also URL-first.

```text
User pastes source/portal URL
      ↓
App extracts domain
      ↓
Source saves immediately
      ↓
Default isPortal = false
      ↓
User can check/toggle isPortal anytime
```

Example pasted source URL:

```text
https://remoteok.com/remote-dev-jobs
```

Extracted domain:

```text
remoteok.com
```

Saved source:

```js
{
  id: "uuid",
  domain: "remoteok.com",
  sourceUrl: "https://remoteok.com/remote-dev-jobs",
  displayName: "remoteok.com",
  isPortal: false,
  createdAt: "...",
  updatedAt: "..."
}
```

## Updating `isPortal`

On the Sources Page, every source has an `isPortal` checkbox/toggle.

```text
User checks isPortal
      ↓
Source updates immediately
      ↓
updatedAt changes
```

This allows any source to be marked as a portal later.

Examples:

```text
linkedin.com   → isPortal = true
remoteok.com   → isPortal = true
facebook.com   → isPortal = false
company.com    → isPortal = false
```

---

# Duplicate Rule

```text
If same jobUrl already exists:
  do not save again
  show “Already saved”
```

---

# UI Pages

## Tracker Page

```text
[ Paste job link ]

Jobs list:
- source domain
- job link
- status dropdown
- delete button
- search/filter
```

## Sources Page

```text
Sources list:
- display name
- domain
- isPortal toggle
- add source
- edit source
- delete unused source
```

## Backup Page

```text
Export JSON
Import JSON
```

---

# Export Format

```js
{
  version: 1,
  exportedAt: "...",
  jobs: [],
  sources: []
}
```

