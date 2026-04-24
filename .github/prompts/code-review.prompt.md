---
description: Review a local diff, commit, or PR branch with a severity-tagged report covering correctness, security, performance, readability, and test coverage.
agent: 'agent'
tools: ["read", "execute"]
---

# Code Review

Produce a concise, severity-tagged review of code changes, usually for a local diff before the user pushes or for the latest commit on a checked-out PR branch.

## Goal

Identify the highest-signal issues in a code change and present them in a review format the author can act on quickly.

## Inputs

Use the user's stated scope when provided. If the user does not specify scope, default to reviewing the last commit with `git show HEAD` and state that assumption briefly.

## Steps

1. Determine the review scope:
   - Last commit: `git show HEAD`
   - Last commit diff only: `git diff HEAD~1 HEAD`
   - Unpushed commits: `git diff @{upstream}..HEAD`
   - Working tree: `git diff` and `git diff --staged`
   - Full PR branch vs target: `git diff <target>...HEAD`
2. Inspect the diff and read surrounding code only when needed to verify a suspected issue.
3. Evaluate the change across these dimensions:
   - Bugs and correctness
   - Security
   - Performance
   - Style and readability
   - Test coverage
4. Assign one severity tag to each finding:
   - `🔴 critical`
   - `🟡 major`
   - `🔵 minor`
5. Write the review using the report format below.

## Output Format

Use this structure:

```markdown
## Review Summary

<1-3 sentence overall take>

**Scope reviewed:** <scope>
**Counts:** 🔴 <N> critical · 🟡 <N> major · 🔵 <N> minor

---

## Comments

### 🔴 Critical

**`path/to/file.ext:LINE`** — <short title>
<why this matters>

**Suggested fix:**
~~~language
<concrete fix>
~~~

### 🟡 Major

...

### 🔵 Minor

...

---

## What's good

- <noteworthy positive>
```

Omit empty severity sections. If the change is clean, say so plainly and skip the comments section.

## Rules

- Be specific and cite exact files and lines from the diff
- Explain why an issue matters, not just what is different
- Do not invent problems when the evidence is unclear
- Focus on changed code unless the diff depends directly on nearby context
- For very large diffs, prioritize the highest-risk files first and say what you skipped
- If the user supplied a Bitbucket PR URL and fetching it is not possible, fall back to local git on the checked-out branch
