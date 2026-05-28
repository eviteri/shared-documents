---
name: jira-cloud-api
description: Jira Cloud REST API v3 endpoint reference for issues, comments, transitions, search (JQL), and sprints. Use this when the Jira agent needs to make HTTP calls.
---

# Jira Cloud REST API v3

Base path: `{JIRA_BASE_URL}/rest/api/3`
Agile path (sprints/boards): `{JIRA_BASE_URL}/rest/agile/1.0`

All requests use HTTP Basic auth: `Authorization: Basic base64(EMAIL:API_TOKEN)`.

## Issues

### Get issue by key or id

```
GET /rest/api/3/issue/{issueIdOrKey}
```

Useful query params:
- `fields=summary,description,status,assignee,priority,labels,issuetype,parent,customfield_10020` — limit returned fields (cuts payload, faster)
- `expand=renderedFields,changelog,transitions` — get HTML-rendered description, history, available transitions
- `fields=*all` — everything (avoid in production)

Response shape (abbreviated):

```json
{
  "id": "10001",
  "key": "PROJ-123",
  "fields": {
    "summary": "Story title",
    "status": { "name": "In Progress", "statusCategory": { "key": "indeterminate" } },
    "assignee": { "accountId": "5b10...", "displayName": "Alice" },
    "priority": { "name": "Medium" },
    "labels": ["frontend", "tech-debt"],
    "issuetype": { "name": "Story" },
    "description": { "type": "doc", "version": 1, "content": [...] }
  }
}
```

### Update issue fields

```
PUT /rest/api/3/issue/{issueIdOrKey}
```

Body:

```json
{
  "fields": {
    "summary": "New title",
    "labels": ["frontend", "ready-for-review"],
    "priority": { "name": "High" },
    "assignee": { "accountId": "5b10..." },
    "duedate": "2026-05-15",
    "description": { "type": "doc", "version": 1, "content": [/* ADF */] }
  }
}
```

For custom fields, use the field ID (`customfield_10020`), not the human name. Discover IDs via `GET /rest/api/3/field`.

Returns `204 No Content` on success.

### Delete issue (DO NOT USE — out of scope)

The agent must refuse `DELETE /rest/api/3/issue/{key}`.

## Comments

### List comments

```
GET /rest/api/3/issue/{issueIdOrKey}/comment
```

### Add a comment

```
POST /rest/api/3/issue/{issueIdOrKey}/comment
Content-Type: application/json
```

Body — comment body MUST be ADF (see `jira-adf` skill):

```json
{
  "body": {
    "type": "doc",
    "version": 1,
    "content": [
      {
        "type": "paragraph",
        "content": [{ "type": "text", "text": "Comment text here." }]
      }
    ]
  }
}
```

Returns `201` with the created comment object including `self`, `id`, `author`, `created`.

## Transitions (status changes)

### List available transitions

```
GET /rest/api/3/issue/{issueIdOrKey}/transitions
```

Response:

```json
{
  "transitions": [
    { "id": "21", "name": "In Progress", "to": { "name": "In Progress" } },
    { "id": "31", "name": "Done", "to": { "name": "Done" } }
  ]
}
```

Only transitions valid from the current status are returned. If the target status is missing, the workflow forbids that move.

### Execute a transition

```
POST /rest/api/3/issue/{issueIdOrKey}/transitions
```

Body:

```json
{
  "transition": { "id": "31" }
}
```

Returns `204 No Content`. To set fields during the transition (e.g., resolution on Done):

```json
{
  "transition": { "id": "31" },
  "fields": { "resolution": { "name": "Done" } }
}
```

## Search (JQL)

### New endpoint (preferred)

```
POST /rest/api/3/search/jql
```

Body:

```json
{
  "jql": "project = PROJ AND sprint in openSprints() AND statusCategory != Done",
  "fields": ["summary", "status", "assignee", "updated"],
  "nextPageToken": null,
  "maxResults": 50
}
```

Pagination uses `nextPageToken` (token-based, not offset-based as in the legacy endpoint).

### Legacy endpoint (deprecated)

`GET /rest/api/3/search` — still works on many tenants but is being phased out. Prefer `/search/jql`. If you must use it:

```
GET /rest/api/3/search?jql=project%20%3D%20PROJ&fields=summary,status&maxResults=50&startAt=0
```

### Approximate count

```
POST /rest/api/3/search/approximate-count
```

Body: `{ "jql": "..." }` — fast count without materializing results.

## Users

### Find users (for assignment)

```
GET /rest/api/3/user/search?query=alice@company.com
```

Use the returned `accountId` when assigning. **Never** use `name` or `username` — those are deprecated and removed in Cloud.

### Current user

```
GET /rest/api/3/myself
```

## Fields

### List all fields (system + custom)

```
GET /rest/api/3/field
```

Use this once at startup if you need to map human field names ("Story Points") to custom field IDs (`customfield_10016`).

## Projects

### Get project metadata

```
GET /rest/api/3/project/{projectIdOrKey}
```

### List versions

```
GET /rest/api/3/project/{projectIdOrKey}/versions
```

## Sprints (Agile API)

### Get sprints for a board

```
GET /rest/agile/1.0/board/{boardId}/sprint?state=active
```

States: `active`, `closed`, `future`.

### Get issues in a sprint

```
GET /rest/agile/1.0/sprint/{sprintId}/issue?fields=summary,status,assignee,updated
```

### Find boards for a project

```
GET /rest/agile/1.0/board?projectKeyOrId=PROJ
```

## Common HTTP responses

| Status | Meaning | What to tell the user |
|--------|---------|-----------------------|
| `200` / `201` / `204` | Success | Report what changed |
| `400` | Bad request — usually invalid field or ADF | Show the response body verbatim |
| `401` | Auth failed | API token rejected — ask the user to verify `.env` |
| `403` | Forbidden | Account lacks permission on this issue/project |
| `404` | Not found | Issue key wrong, or account has no view permission |
| `429` | Rate limited | Back off, respect `Retry-After` header |

## Rate limits

Jira Cloud applies dynamic rate limits per tenant. On `429`, read the `Retry-After` header (seconds) and pause. Don't retry tight loops.
