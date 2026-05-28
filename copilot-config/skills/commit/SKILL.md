---
name: commit
description: Conventional commit message standards and git commit best practices. Use this when creating commits, writing commit messages, or preparing staged changes.
---

# Commit Standards

## Conventional Commit Format

```
type(scope): subject

body

footer
```

### Types

| Type       | Use for                                  |
| ---------- | ---------------------------------------- |
| `feat`     | New feature or capability                |
| `fix`      | Bug fix                                  |
| `docs`     | Documentation changes only               |
| `refactor` | Code restructure with no behavior change |
| `test`     | Adding or updating tests                 |
| `chore`    | Build, tooling, dependency updates       |
| `perf`     | Performance improvements                 |
| `ci`       | CI/CD configuration changes              |

### Rules

- **Subject line:** imperative mood, ≤72 chars, no period at end ("Add feature" not "Added feature")
- **Body:** explain _why_, not _what_ — the diff already shows what changed
- **Footer:** reference issues (`Closes #123`), breaking changes (`BREAKING CHANGE: ...`)
- **Scope:** optional, names the affected module or area (e.g., `feat(auth): add OAuth login`)

## Staging Rules

- Never `git add .` — stage only files relevant to the change
- Split unrelated changes into separate commits
- Verify staged files with `git diff --staged` before committing

## Examples

```bash
# Feature
git commit -m "feat(auth): add OAuth login via GitHub" -m "Replaces the username/password flow with GitHub OAuth to reduce friction for new users."

# Bug fix
git commit -m "fix(api): handle null response from Strapi when collection is empty" -m "Previously threw a TypeError when data was null. Now returns an empty array."

# Docs only
git commit -m "docs: update README with local setup instructions"
```
