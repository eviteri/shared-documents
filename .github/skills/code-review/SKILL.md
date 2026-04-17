---
name: code-review
description: Use for code reviews, reviewing pull requests, reviewing local changes before pushing, checking a diff, reviewing the last commit, or reviewing a Bitbucket PR pulled down locally. Produces a severity-tagged review covering bugs, security, performance, readability, and test coverage. Trigger for phrases like "review my changes", "review my PR", "check my diff", "review this commit", "what's wrong with this code", "before I push", "before I open the PR", or when the user shares git diff output, a patch file, or a Bitbucket PR URL.
---

# Code Review

Produce a concise, severity-tagged review of code changes — typically a local diff before the user pushes, or the latest commit on a Bitbucket PR branch they've pulled down.

## When to use this skill

Trigger for any request to review code changes: pre-push sanity checks, reviewing the last commit, checking a diff, or reviewing a Bitbucket PR that's been pulled down locally. Don't wait for explicit phrasing — "can you look at my changes?" or "about to push this, anything look off?" both qualify.

## Two primary modes

**Mode A: Local pre-submit review** (most common). User is about to push or open a PR.

**Mode B: Downloaded Bitbucket PR review.** User has checked out a PR branch and wants to review what changed — usually the latest commit if the author just pushed an update.

If the user doesn't specify a scope, default to **the last commit** (`git show HEAD`) and confirm briefly: "Reviewing your last commit — let me know if you meant something else."

## Getting the diff

Run the appropriate git command yourself. Pick the one that matches the user's intent:

| Intent | Command |
|---|---|
| Review the last commit | `git show HEAD` |
| Review the last commit (diff only, no message) | `git diff HEAD~1 HEAD` |
| Review all unpushed commits | `git diff @{upstream}..HEAD` |
| Review uncommitted working tree | `git diff` and `git diff --staged` |
| Review the whole PR vs the target branch | `git diff <target>...HEAD` (usually `main`, `master`, or `develop`) |
| Review latest commit on a downloaded PR | `git show HEAD` |

For Bitbucket PR URLs, the raw diff is at `https://bitbucket.org/<workspace>/<repo>/pull-requests/<N>/diff` (Cloud) or `<host>/projects/<KEY>/repos/<slug>/pull-requests/<N>.diff` (Server/Data Center). Most Bitbucket repos are private, so fetching the URL will usually fail — if it does, fall back to running git locally on the checked-out branch.

If the diff is empty or ambiguous, ask the user to clarify what they want reviewed before proceeding.

## Review dimensions

Evaluate across all five. Skip a dimension in the output if it has no issues — don't write "no issues found" for every category.

1. **Bugs & correctness** — off-by-ones, null/undefined handling, race conditions, unhandled promise rejections, incorrect error handling, broken edge cases, type mismatches, regressions of existing behavior, incorrect equality checks.
2. **Security** — hardcoded secrets/tokens/keys, SQL injection, command injection, XSS, SSRF, path traversal, missing input validation, unsafe deserialization, insecure crypto (MD5, SHA-1, ECB mode), broken auth/authz, IDOR, missing permission checks.
3. **Performance** — N+1 queries, missing indexes implied by new queries, O(n²) where O(n) is feasible, unnecessary allocations in hot paths, blocking calls in async code, unbounded loops/recursion.
4. **Style & readability** — unclear names, dead code, inconsistent formatting with the surrounding file, overly complex functions, missing or misleading comments, violations of the project's apparent conventions.
5. **Test coverage** — new behavior without tests, tests that don't actually assert the new behavior, missing edge-case tests, tests coupled to implementation details.

## Severity tags

Every comment gets exactly one tag. Be disciplined — inflating severity makes the review less useful.

- **🔴 critical** — will cause incorrect behavior in production, a security hole, data loss, or broken builds. Must fix before pushing/merging.
- **🟡 major** — likely to cause problems (edge cases, performance regressions, missing tests for important paths), or significantly hurts maintainability. Should fix before the PR goes out.
- **🔵 minor** — style, naming, small readability improvements, nits. Safe to push without fixing.

If you're debating between two levels, pick the lower one. Most diffs have zero or one critical issue.

## Output format

Use this exact structure:

```
## Review Summary

<1–3 sentence overall take. What the changes do, overall quality, and whether they're ready to push, need changes, or have blocking issues.>

**Scope reviewed:** <e.g., "last commit (abc1234)", "working tree vs main", "latest commit on PR branch">
**Counts:** 🔴 <N> critical · 🟡 <N> major · 🔵 <N> minor

---

## Comments

### 🔴 Critical

**`path/to/file.ext:LINE`** — <short title>
<1–3 sentences explaining the issue and why it matters. Reference the specific code.>

**Suggested fix:**
```<language>
<concrete code, or a concise description if code isn't the right medium>
```

<repeat for each critical issue>

### 🟡 Major

<same format>

### 🔵 Minor

<same format — "Suggested fix" block is optional if the fix is obvious>

---

## What's good

<1–3 bullets on genuinely noteworthy positives. Skip the section entirely if nothing stands out; don't pad.>
```

Omit any severity section with zero comments. If the diff is clean across the board, say so plainly in the summary and skip the Comments section.

## Style rules for comments

- **Be specific.** Exact file paths and line numbers from the diff. "`auth.py:47`" beats "the auth code."
- **Explain the "why," not just the "what."** "This can throw on empty input, which happens when users submit the form without a file" beats "handle empty input."
- **Suggest, don't command.** "Consider using a `Set` here to dedupe in O(n)" beats "Use a Set."
- **No filler.** Skip "Great work overall!" preambles and "Hope this helps!" closers.
- **Don't repeat yourself.** If the same issue appears in multiple places, comment once and note "same pattern at lines X, Y, Z."
- **Don't invent problems.** If you're not sure something is wrong, read more of the surrounding code to verify, or skip it. False positives erode trust fast.
- **Don't comment on code outside the diff** unless a change in the diff directly depends on surrounding code that's broken.
- **Pre-submit framing:** Phrase fixes as "before you push" / "before you open the PR" rather than "before merge," since this is usually pre-push.

## Workflow

1. **Determine scope.** Local uncommitted? Last commit? Full PR vs target branch? Latest commit on a downloaded PR? If unclear, default to last commit and confirm.
2. **Get the diff.** Run the appropriate git command. If the user provided a URL, try fetching it; fall back to local git on failure.
3. **Scan for context.** Note the languages, frameworks, and project conventions visible in the diff (naming style, test framework, error handling patterns). Tailor feedback to match the existing code.
4. **First pass** across all five dimensions. Collect candidate issues.
5. **Prune.** For each candidate, ask: is this actually wrong, or am I pattern-matching? Drop uncertain ones unless you can verify by reading more code.
6. **Assign severities** using the rubric above.
7. **Write the review** in the exact output format. Keep each comment tight.

## Edge cases

- **Huge diffs (>1000 lines):** Focus on the highest-risk files first (auth, data models, anything touching money or PII, concurrency). State in the summary that you prioritized and what you skipped.
- **Generated code / lockfiles / vendored deps:** Skip these. Mention in the summary if they dominated the diff.
- **Pure refactors:** Focus on whether behavior actually stays identical. Correctness beats style here.
- **Docs-only changes:** Review for accuracy, clarity, and broken links. Severity caps at major unless docs describe a security/safety procedure incorrectly.
- **Commit messages:** When reviewing `git show HEAD`, the message is included. Briefly flag if it's unclear or misleading, but don't dwell on it.
- **WIP / obviously unfinished code:** Focus on direction and approach, not nitpicks on incomplete sections.
- **Can't access a Bitbucket PR URL:** Expected — most Bitbucket repos are private. Ask the user to `git fetch && git checkout <pr-branch>` and rerun locally.

## Example

**User says:** "review my last commit"

**Skill runs:** `git show HEAD` — sees a new endpoint that takes a user-supplied filename and reads it from disk.

**Output:**

```
## Review Summary

Adds a `/download` endpoint that streams a file by name. Core logic works, but there's a path traversal vulnerability that must be fixed before you push, and the new code has no tests.

**Scope reviewed:** last commit (abc1234)
**Counts:** 🔴 1 critical · 🟡 2 major · 🔵 1 minor

---

## Comments

### 🔴 Critical

**`routes/download.py:23`** — Path traversal via unvalidated filename
The handler passes `request.args["name"]` directly to `open()`. A request for `?name=../../etc/passwd` will read arbitrary files off the server. Exploitable by any unauthenticated user.

**Suggested fix:**
```python
from pathlib import Path
base = Path("/var/app/downloads").resolve()
target = (base / request.args["name"]).resolve()
if not target.is_relative_to(base):
    abort(400)
```

### 🟡 Major

**`routes/download.py:19`** — No tests for the new endpoint
Happy path, missing-file, and (once fixed) traversal-attempt cases should all be covered before the PR goes out. `tests/test_routes.py` has the fixtures you'd need.

**`routes/download.py:31`** — File handle leaked on exception
`open()` without a `with` block will leak the descriptor if `send_file` raises. Use a context manager or `flask.send_from_directory`.

### 🔵 Minor

**`routes/download.py:15`** — Function name `dl` is terse
`download_file` would match the style of other handlers in this file (`upload_file`, `list_files`).
```
