---
name: code-review
description: Code review standards and severity guidelines. Use this when reviewing code changes, evaluating diffs, or assessing pull request quality.
---

# Code Review Standards

## Severity Levels

| Tag           | When to use                                                                                |
| ------------- | ------------------------------------------------------------------------------------------ |
| `🔴 critical` | Bugs, security vulnerabilities, data loss risks, broken functionality                      |
| `🟡 major`    | Performance issues, logic errors, missing test coverage for key paths, poor error handling |
| `🔵 minor`    | Style inconsistencies, readability improvements, non-critical refactors                    |

## Review Dimensions

### Correctness

- Does the logic match the intended behavior?
- Are edge cases (null, empty, overflow) handled?
- Are error states and rejections handled properly?

### Security

- No hardcoded secrets, tokens, or credentials
- Input is validated and sanitized at system boundaries
- No SQL injection, XSS, or path traversal risks
- Auth checks are present where required

### Performance

- No unnecessary loops inside loops (O(n²) when O(n) is possible)
- No blocking calls in hot paths
- Resources (files, connections) are properly closed

### Readability

- Variable and function names are clear and consistent
- Complex logic has inline explanation
- Dead code is removed

### Test Coverage

- New functions have at least one test
- Edge cases identified in the review are covered
- Existing tests still pass against the change

## Output Format

```markdown
## Review Summary

<1-3 sentence overall take>

**Scope reviewed:** <scope>
**Counts:** 🔴 <N> critical · 🟡 <N> major · 🔵 <N> minor

---

## Comments

### 🔴 Critical

**`path/to/file.ext:LINE`** — <short title>
<why this matters and what to do>

### 🟡 Major

**`path/to/file.ext:LINE`** — <short title>
<why this matters and what to do>

### 🔵 Minor

**`path/to/file.ext:LINE`** — <short title>
<suggestion>
```
