---
name: jira-jql
description: JQL (Jira Query Language) syntax reference for filtering and searching Jira issues. Use this when constructing queries for the search endpoint or for stale-work detection.
---

# JQL — Jira Query Language

JQL is used in `POST /rest/api/3/search/jql` and across Jira UIs. It is **not SQL** — no joins, no `SELECT`. You only filter and sort.

## Structure

```
<field> <operator> <value> [AND|OR] ... [ORDER BY <field> ASC|DESC]
```

## Operators

| Operator | Use |
|----------|-----|
| `=` `!=` | Exact match |
| `>` `>=` `<` `<=` | Numeric / date comparison |
| `IN (...)` `NOT IN (...)` | Match any of a list |
| `~` | Contains (text fields, fuzzy) |
| `!~` | Does not contain |
| `IS EMPTY` / `IS NOT EMPTY` | Null check |
| `WAS` / `CHANGED` | History queries (e.g., `status WAS "In Progress" BEFORE -7d`) |

## Common fields

| Field | Example |
|-------|---------|
| `project` | `project = PROJ` or `project = "My Project"` |
| `issuetype` | `issuetype in (Story, Bug)` |
| `status` | `status = "In Progress"` |
| `statusCategory` | `statusCategory != Done` (categories: `new`, `indeterminate`, `done`) |
| `assignee` | `assignee = currentUser()` or `assignee is EMPTY` |
| `reporter` | `reporter = "alice@company.com"` |
| `priority` | `priority in (High, Highest)` |
| `labels` | `labels = "tech-debt"` |
| `sprint` | `sprint in openSprints()` |
| `created` / `updated` / `resolved` / `duedate` | `updated < -14d` |
| `text` | `text ~ "checkout flow"` (full-text search across summary, description, comments) |
| `summary` / `description` / `comment` | `summary ~ "login"` |
| `parent` | `parent = PROJ-100` (epic link) |
| `fixVersion` | `fixVersion = "v1.2.0"` |
| `resolution` | `resolution = Unresolved` |

## Date math

Use relative offsets — Jira interprets these from "now":

| Expression | Meaning |
|------------|---------|
| `-1d` | 1 day ago |
| `-2w` | 2 weeks ago |
| `-1M` | 1 month ago |
| `"2026-01-15"` | Absolute date |
| `startOfDay()` `endOfWeek()` `startOfMonth()` | Boundary functions |

Examples:

```
updated < -14d
created >= startOfMonth(-1) AND created < startOfMonth()
duedate < endOfWeek()
```

## Useful functions

| Function | Use |
|----------|-----|
| `currentUser()` | The authenticated user |
| `openSprints()` | All currently active sprints |
| `closedSprints()` | Completed sprints |
| `futureSprints()` | Planned sprints |
| `unreleasedVersions()` / `releasedVersions()` | Version filters |
| `issueHistory()` | Recently viewed |
| `linkedIssues(KEY)` | Issues linked to KEY |

## Quoting

- Quote values with spaces or special chars: `status = "In Progress"`
- Quote project names if not using the key: `project = "My Project"`
- Reserved words must be quoted as field values
- Custom field display names with spaces: `"Story Points" >= 5` (or use `cf[10016] >= 5`)

## Patterns the Jira agent uses

### Stale stories in active sprint

```
project = "PROJ" AND sprint in openSprints() AND statusCategory != Done AND updated < -14d ORDER BY updated ASC
```

### Unassigned work

```
project = "PROJ" AND assignee is EMPTY AND statusCategory != Done
```

### Stories without acceptance criteria (heuristic — no description)

```
project = "PROJ" AND issuetype = Story AND description is EMPTY
```

### Carryover from last sprint

```
project = "PROJ" AND sprint in closedSprints() AND sprint in openSprints() AND statusCategory != Done
```

(An issue in both a closed and an open sprint = it carried over.)

### My in-progress work

```
assignee = currentUser() AND statusCategory = "In Progress" ORDER BY priority DESC, updated ASC
```

### Recently changed by someone

```
project = "PROJ" AND status CHANGED BY "alice@company.com" AFTER -7d
```

### Blocked work

```
project = "PROJ" AND (labels = blocked OR status = Blocked OR text ~ "blocked by")
```

## Sort

`ORDER BY` goes at the end. Multiple fields allowed:

```
ORDER BY priority DESC, updated ASC
```

## Validation

Before sending a JQL query to the API, sanity-check:

- All quoted strings have matching quotes
- Field names are real (use `GET /rest/api/3/field` if unsure)
- Custom fields use either the display name (in quotes) or `cf[XXXXX]`
- Date offsets use lowercase units except `M` for month (lowercase `m` = minute)

If a query returns 400, the response body usually pinpoints the bad token — surface it to the user verbatim.
