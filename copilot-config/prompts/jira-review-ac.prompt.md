---
description: Review a Jira story for missing or weak acceptance criteria and propose concrete fixes.
agent: "Jira Agent"
tools: ["read", "web"]
---

Review a Jira story's acceptance criteria and propose specific improvements. Do NOT modify the issue — propose only.

## Steps

1. **Fetch the issue** — `GET /rest/api/3/issue/{KEY}?fields=summary,description,issuetype,labels&expand=renderedFields`.

2. **Locate AC** — scan the description for any of:
   - Heading "Acceptance Criteria" / "AC" / "Definition of Done"
   - "Given … When … Then …" blocks
   - Numbered lists or checklists after a goal/context section

3. **Evaluate against this rubric**:

   | Check | Pass condition |
   |-------|----------------|
   | Present at all | AC section exists |
   | Testable | Each AC describes an observable outcome (not "works", "looks good") |
   | Specific | Concrete inputs, states, or thresholds (not "fast", "improved") |
   | Edge cases | Errors, empty states, permissions, loading, offline addressed |
   | Negative cases | What should NOT happen is stated |
   | Measurable | Performance/UX claims have numbers (e.g., "<200ms p95") |
   | Unambiguous | No "we should consider", no TODOs, no "TBD" |
   | Scoped | One story = one cohesive outcome — not a grab bag |

4. **Output a punch list**:

   ```
   ## AC Review — {KEY}

   **Verdict:** {Strong / Adequate / Needs work / Missing}

   **Issues found:**
   1. {Specific problem, quoting the offending text}
      → Suggested rewrite: "{concrete replacement}"
   2. ...

   **Missing coverage:**
   - {Edge case or scenario not addressed, e.g., "What happens when the user is logged out?"}

   **Recommended additions:**
   - Given {context}, when {action}, then {observable outcome}
   - ...
   ```

5. **Offer next steps** — at the end, ask:

   > Would you like me to (a) post this review as a comment on {KEY}, or (b) update the description directly with the suggested AC?

   Wait for the user to confirm before doing either. If they say "post comment", follow the Jira Agent's inline **Add a comment** workflow. If they say "update description", follow the Jira Agent's inline **Update issue fields** workflow.

6. **Refuse** to auto-edit on the basis of this review without explicit confirmation.
