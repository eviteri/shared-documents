---
description: Detect stale stories in the active sprint and explain why each one might be stuck.
agent: "Jira Agent"
tools: ["read", "web"]
---

Find stale work in active sprints for a project and produce an actionable list.

## Steps

1. **Resolve credentials and project key**. The project key is dynamic. Derive it from any issue key in the user message (`FSLTL-1053` → `FSLTL`). If the user only said "stale stories" with no issue key and no prior project in context, ask: "Which project key?".

2. **Resolve the staleness threshold** — default `14` days. The user can override in their message ("stale = older than 7 days").

3. **Run JQL search** via the new endpoint:

   ```
   POST {JIRA_BASE_URL}/rest/api/3/search/jql
   Content-Type: application/json

   {
     "jql": "project = \"{KEY}\" AND sprint in openSprints() AND statusCategory != Done AND updated < -{staleDays}d ORDER BY updated ASC",
     "fields": ["summary", "status", "assignee", "updated", "labels", "priority", "comment"],
     "maxResults": 100
   }
   ```

4. **Enrich each result** with a stuck-hypothesis. For each issue, infer ONE most likely reason:

   | Signal | Hypothesis |
   |--------|------------|
   | `assignee` is null | "Unassigned — needs an owner" |
   | Status is `In Review` | "Awaiting review — ping reviewer" |
   | Label includes `blocked` or `waiting` | "Self-tagged as blocked" |
   | Last comment > {staleDays}d ago AND status = `In Progress` | "No recent activity — owner may be context-switched" |
   | Status changed many times back-and-forth (use `expand=changelog` if needed) | "Bouncing between states — scope unclear" |
   | Priority `Low` and no parent epic | "Likely deprioritized — consider removing from sprint" |

5. **Output a compact table**:

   ```
   ## Stale stories in {KEY} — open sprints (threshold: {staleDays}d)

   {N} stale stories found.

   | Key | Summary | Status | Assignee | Idle | Why |
   |-----|---------|--------|----------|------|-----|
   | PROJ-123 | Add CSV export | In Review | Alice | 21d | Awaiting review — ping reviewer |
   | PROJ-145 | Refactor auth | In Progress | — | 18d | Unassigned — needs an owner |
   ```

   Sort by idle days descending.

6. **Top recommendations** — after the table, give 2–3 concrete next actions:

   ```
   Suggested next actions:
   1. Reassign PROJ-145 or remove from sprint
   2. Ping reviewer on PROJ-123 (idle 21d in review)
   3. Hold a 15-min sprint hygiene huddle if 5+ stories are idle
   ```

7. **Edge cases**:
   - Zero results → "No stale stories — sprint is healthy by the {staleDays}-day threshold"
   - No active sprint → "No open sprint found for project {KEY}"
   - Project has no Scrum board → suggest using a `updated < -{staleDays}d` filter without `openSprints()`
