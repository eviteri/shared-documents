---
name: jira-workflows
description: Step-by-step procedures for the most common Jira write and read flows — fetching, summarizing, commenting, updating fields, and transitioning issues. Use this whenever the Jira agent needs to perform any of these operations.
---

# Jira Workflows

Procedural recipes the Jira agent uses when handling natural-language requests like "give me FSLTL-1053", "comment on PROJ-12 saying X", "move FSLTL-1053 to In Review", "set the priority to High".

All flows assume:
- Credentials loaded from environment variables or the repository-root `.env` (`JIRA_BASE_URL`, `JIRA_EMAIL`, `JIRA_API_TOKEN`)
- Auth header: `Authorization: Basic base64(JIRA_EMAIL:JIRA_API_TOKEN)`
- Project key is derived from the issue key in the user's message (`FSLTL-1053` → `FSLTL`)

For endpoint-level reference see `jira-cloud-api`. For ADF body construction see `jira-adf`. For JQL syntax see `jira-jql`.

---

## Fetch a story

Trigger phrases: "give me {KEY}", "show me {KEY}", "what's {KEY}", "details for {KEY}".

### Steps

1. Extract the issue key from the message with regex `[A-Z][A-Z0-9]+-\d+`.
2. Call:
   ```
   GET {JIRA_BASE_URL}/rest/api/3/issue/{KEY}?fields=summary,description,status,assignee,reporter,priority,labels,issuetype,parent,duedate,created,updated&expand=renderedFields
   ```
3. Render a structured block:
   ```
   {KEY} — {summary}
   Status:    {status.name}  ({statusCategory})
   Type:      {issuetype.name}
   Priority:  {priority.name}
   Assignee:  {assignee.displayName or "Unassigned"}
   Reporter:  {reporter.displayName}
   Labels:    {labels joined with ", " or "—"}
   Parent:    {parent.key} — {parent.fields.summary} (if any)
   Due:       {duedate or "—"}
   Updated:   {updated} ({N days ago})
   URL:       {JIRA_BASE_URL}/browse/{KEY}

   Description:
   {description as plain text — walk the ADF tree, or use renderedFields.description and strip HTML}
   ```
4. Errors:
   - `404` → "Issue {KEY} not found, or your account can't view it"
   - `401` → "Auth failed — check `.env` credentials"
   - `403` → "No permission to read this issue"

---

## Summarize a story

Trigger phrases: "summarize {KEY}", "TL;DR on {KEY}", "what's {KEY} about?".

### Steps

1. Fetch the issue (same call as above, include `comment` in fields).
2. Read the description as plain text.
3. Output (under 200 words):
   ```
   {KEY} — {one-line TL;DR}

   **Goal:** {user-facing outcome in plain English}

   **Status:** {status} • {assignee} • due {duedate or "—"}

   **In scope:**
   - {bullet}

   **Out of scope / open questions:**
   - {bullet}

   **Acceptance criteria detected:** {yes — list them / no — flag it}

   **Recent activity:** {last comment author + date + 1-line gist, if any}
   ```
4. Heuristics:
   - Empty description → say so and stop after the metadata
   - Vague AC → flag, but don't propose fixes here (suggest `/jira-review-ac`)
   - Description >500 words or >10 ACs → note it might benefit from splitting
5. Tone: neutral, factual. No marketing voice.

---

## Add a comment

Trigger phrases: "comment on {KEY} saying...", "post a note on {KEY}...", "leave feedback on {KEY}".

### Steps

1. Extract the issue key. If the user didn't supply text, ask what to write.
2. Build the comment body in ADF (see `jira-adf`):
   - Plain text → single paragraph
   - Bullet-style markdown → `bulletList`
   - Triple-backtick code → `codeBlock`
   - `**bold**` → text node with `strong` mark
   - `@<email or name>` → resolve via `GET /rest/api/3/user/search?query=<value>` and emit a `mention` node with the returned `accountId`
3. Show a preview and **wait for confirmation**:
   ```
   About to post on {KEY} ({summary}):
   ────────────────────────────────────
   {comment as plain text}
   ────────────────────────────────────
   Confirm? (yes / no / edit)
   ```
4. On `yes`:
   ```
   POST {JIRA_BASE_URL}/rest/api/3/issue/{KEY}/comment
   Content-Type: application/json

   { "body": { "type": "doc", "version": 1, "content": [...] } }
   ```
5. Report:
   - `201` → "Comment posted → {JIRA_BASE_URL}/browse/{KEY}?focusedCommentId={id}"
   - `400` → surface the body verbatim (usually malformed ADF)
   - `403` → "No permission to comment on this issue"
6. **Refuse** to skip the confirmation step.

---

## Update issue fields

Trigger phrases: "set {KEY} priority to High", "assign {KEY} to alice@...", "label {KEY} as ...", "change the title of {KEY} to ...", "set due date on {KEY} to ...".

### Steps

1. Extract the issue key. Fetch current values:
   ```
   GET {JIRA_BASE_URL}/rest/api/3/issue/{KEY}?fields=*all
   ```
2. Map the user's intent to field names:

   | User says | Field | Value shape |
   |---|---|---|
   | "title", "summary" | `summary` | string |
   | "description" | `description` | ADF doc |
   | "labels", "tags" | `labels` | `["a", "b"]` |
   | "priority" | `priority` | `{ "name": "High" }` |
   | "assignee", "assign to" | `assignee` | `{ "accountId": "..." }` |
   | "due date" | `duedate` | `"YYYY-MM-DD"` |
   | "story points" | `customfield_10016` (verify via `GET /rest/api/3/field`) | number |
   | "epic", "parent" | `parent` | `{ "key": "PROJ-100" }` |

   For unknown custom-field names, fetch `GET /rest/api/3/field` and grep for the human label.

3. **Resolve assignee** — when the user gives an email or display name, look up the `accountId` first:
   ```
   GET {JIRA_BASE_URL}/rest/api/3/user/search?query=<email or name>
   ```
   If multiple users match, list them and ask the user to pick. To unassign, send `"assignee": null`.

4. Show a diff and **wait for confirmation**:
   ```
   Updating {KEY} ({summary}):
     summary:   "Old title" → "New title"
     priority:  Medium → High
     labels:    [frontend] → [frontend, ready-for-review]
     assignee:  Alice → Bob (5b10...)

   Confirm? (yes / no / edit)
   ```

5. On `yes`, send only the changed fields:
   ```
   PUT {JIRA_BASE_URL}/rest/api/3/issue/{KEY}
   Content-Type: application/json

   { "fields": { ...only changed fields... } }
   ```

6. Report:
   - `204` → "Updated {KEY} → {JIRA_BASE_URL}/browse/{KEY}"
   - `400` → surface the body (typical: invalid priority name, unknown custom field, malformed ADF, stale accountId)
   - `403` → "No permission to edit this issue"

7. **Refuse** to skip the diff/confirmation step.

---

## Transition status

Trigger phrases: "move {KEY} to Done", "transition {KEY} to In Review", "mark {KEY} as Blocked".

### Steps

1. Extract the issue key.
2. Fetch the current state and the available transitions in parallel:
   ```
   GET {JIRA_BASE_URL}/rest/api/3/issue/{KEY}?fields=summary,status
   GET {JIRA_BASE_URL}/rest/api/3/issue/{KEY}/transitions
   ```
3. Match the user's target (case-insensitive) against `transitions[].to.name` and `transitions[].name`:
   - One match → proceed
   - Multiple → list and ask the user to pick by id or name
   - None → tell the user the workflow forbids that move, and list what IS available:
     ```
     {KEY} is currently "In Progress". Available transitions:
       - In Review (id 41)
       - Blocked (id 51)

     "Done" is not reachable from this state.
     ```
4. Confirm before posting:
   ```
   Transition {KEY} ({summary}):
     {currentStatus} → {targetStatus}

   Confirm? (yes / no)
   ```
5. On `yes`:
   ```
   POST {JIRA_BASE_URL}/rest/api/3/issue/{KEY}/transitions
   Content-Type: application/json

   { "transition": { "id": "{transitionId}" } }
   ```
   If the transition screen requires fields (e.g., `resolution` on Done), include them:
   ```
   { "transition": { "id": "31" }, "fields": { "resolution": { "name": "Done" } } }
   ```
6. Report:
   - `204` → "Transitioned {KEY}: {previous} → {new}. {JIRA_BASE_URL}/browse/{KEY}"
   - `400` → surface body (usually a missing required field on the transition screen)
   - `403` → "No permission to transition this issue"

---

## Hand-offs to user-invoked prompts

Some tasks are deliberately user-triggered with `/`-commands:

- `/jira-review-ac` — structured AC audit on a single story
- `/jira-stale-stories` — sprint-wide stale-work scan
- `/jira-suggest-improvements` — engineering-process audit (story / sprint / project)

When a user's natural-language request maps cleanly to one of these, suggest the slash command. If the user clearly asks you to proceed now, run the matching workflow instead of blocking on ceremony.
