---
description: Audit a Jira story, sprint, or project and recommend concrete engineering process improvements.
agent: "Jira Agent"
tools: ["read", "web"]
---

Produce a prioritized, concrete set of recommendations for an engineering team based on Jira data. Scope can be a single story, an active sprint, or a whole project.

## Steps

1. **Determine scope** from the user's message:
   - Single key (e.g., `PROJ-123`) → scope is one story
   - Project key only (e.g., `PROJ`) → scope is the active sprint(s) of that project
   - "Whole project", "last 30 days", or similar → broader audit

2. **Pull data** based on scope.

   ### Single story
   ```
   GET /rest/api/3/issue/{KEY}?fields=*all&expand=renderedFields,changelog
   ```
   Inspect: AC quality (run `jira-review-ac` mentally), description size, label hygiene, assignee, links to PRs, comment activity, time in current status.

   ### Active sprint
   ```
   POST /rest/api/3/search/jql
   { "jql": "project = \"{KEY}\" AND sprint in openSprints()",
     "fields": ["summary","status","assignee","labels","priority","issuetype","updated","duedate","parent"],
     "maxResults": 200 }
   ```

   ### Project audit (last 30 days)
   ```
   POST /rest/api/3/search/jql
   { "jql": "project = \"{KEY}\" AND updated >= -30d",
     "fields": ["summary","status","statusCategory","assignee","labels","priority","issuetype","created","resolved","parent"],
     "maxResults": 500 }
   ```

3. **Run these checks** (all scopes — apply only the ones that have data):

   ### Story-level
   - Missing/weak AC
   - No description or under 1 sentence
   - No assignee
   - No parent epic on a Story
   - "Story" issue type used for a single bug fix or single line change
   - Story open >2 sprints (carryover)

   ### Sprint-level
   - >20% of stories unassigned
   - >30% have no AC
   - Multiple stories with the same vague label (e.g., "improvement", "tech-debt") and no epic grouping
   - Carryover ratio >25% from previous sprint
   - Wide priority distribution within one sprint (everything "High")
   - Bugs not separated from stories (mixed issuetype with no labels)

   ### Project-level
   - Cycle time trend (created → resolved) widening
   - Bug-to-story ratio creeping up
   - Stale labels (used once and never again)
   - Stories without linked PRs/branches (workflow gap)

4. **Output — a prioritized punch list**:

   ```
   ## Improvement recommendations — {scope}

   ### High impact
   1. **{Recommendation title}**
      - Evidence: {data point with issue keys}
      - Action: {concrete step}
      - Owner: {role, e.g., "EM" or "tech lead"}

   ### Medium impact
   2. ...

   ### Quick wins
   3. ...
   ```

   Each recommendation MUST cite specific issue keys or numbers. No vague advice ("communicate better"). If you can't back it with data, drop it.

5. **Tone** — direct, engineer-to-engineer. Not consultant-speak. No filler.

6. **Offer follow-up**:

   > Want me to (a) post this audit as a comment on a tracking issue, (b) draft updates for the worst-offender stories, or (c) export the data?

   Wait for the user to confirm before any write.

7. **Constraints**:
   - Read-only by default — never auto-update or auto-comment from this prompt
   - Don't recommend tooling changes outside Jira unless asked
   - Cap output at the top 10 recommendations — relentlessly prioritize
