---
name: copilot-best-practices-instructions
description: Best practices for creating GitHub Copilot custom instructions (copilot-instructions.md and .instructions.md files). Use this when reviewing, creating, or troubleshooting instruction files.
---

# Copilot Instructions Best Practices

## What are instructions?

Instructions are persistent rules that Copilot follows in every interaction. They define coding standards, conventions, and project-wide constraints. They are always active — users don't invoke them.

## When to use instructions

- The rule applies to **every** Copilot interaction in this repo
- It defines **coding style**, naming conventions, or language preferences
- It sets **project-wide constraints** ("always use TypeScript", "prefer functional components")
- It's **short** and doesn't need tools or special behavior

## When NOT to use instructions

- It only applies to a specific domain or technology → use a **Skill**
- It requires tool access → use an **Agent** or **Prompt**
- It's a task template the user runs on demand → use a **Prompt**
- It would add noise to unrelated interactions → use a **Skill**

## Types and locations

### Repository-wide instructions

- **Location:** `.github/copilot-instructions.md`
- **Scope:** All Copilot interactions in the repo
- **Format:** Plain Markdown (no frontmatter required)

### Path-specific instructions

- **Location:** `.github/instructions/<name>.instructions.md` (supports subdirectories)
- **Scope:** Only when working with files matching the `applyTo` glob
- **Format:** Markdown with required frontmatter

### Agent instructions

- **Location:** `AGENTS.md` at repo root (primary agent instructions)
- **Alternatives:** `CLAUDE.md` or `GEMINI.md` at repo root
- **Also found in:** current working directory, directories in `COPILOT_CUSTOM_INSTRUCTIONS_DIRS`
- **Scope:** Agent-specific behavior
- **Note:** `AGENTS.md` at root is treated as primary instructions; others are additional

### Personal instructions

- **Location:** `$HOME/.copilot/copilot-instructions.md`
- **Scope:** All repos, all interactions for this user

### Environment variable directories

- **Variable:** `COPILOT_CUSTOM_INSTRUCTIONS_DIRS` (comma-separated list of directories)
- **Copilot looks for:** `AGENTS.md` and `.github/instructions/**/*.instructions.md` in each directory

## Path-specific frontmatter

```yaml
---
applyTo: "**/*.py"
---
```

Or with agent exclusion:

```yaml
---
applyTo: "**/*.ts,**/*.tsx"
excludeAgent: code-review
---
```

| Field | Type | Purpose |
|-------|------|---------|
| `applyTo` | string | Glob pattern(s) for matching files (comma-separated) |
| `excludeAgent` | string | Exclude from specific agents (`code-review`, `cloud-agent`, or both) |

### Common glob patterns

| Pattern | Matches |
|---------|---------|
| `*` | All files in current directory |
| `**/*` | All files recursively |
| `*.py` | Python files in current directory |
| `**/*.py` | Python files in all directories |
| `src/**/*.ts` | TypeScript files in src/ recursively |
| `**/*.test.ts` | All test files |

## Body content

Instructions should be:

- **Short and direct** — each rule should be one or two sentences
- **Non-conflicting** — avoid rules that contradict each other
- **Actionable** — tell Copilot what to do, not background information

**Good:**
```markdown
- Use camelCase for variables and PascalCase for React components
- Always use async/await instead of .then() chains
- Write unit tests for all new public functions
```

**Bad:**
```markdown
Our team decided in Q3 2024 that we would standardize on a consistent naming
convention across the codebase after a long discussion about the pros and cons
of different approaches. We eventually settled on camelCase because...
```

## Important behavior

- Instructions take effect **immediately** on save
- When path-specific and repo-wide instructions both exist, **both are applied**
- If instructions **conflict**, Copilot's choice is **non-deterministic** — avoid conflicts
- Whitespace between instructions is ignored

## Checklist

### Repository-wide (`copilot-instructions.md`)

- [ ] File is at `.github/copilot-instructions.md`
- [ ] Rules apply universally to all interactions
- [ ] Each rule is concise (1-2 sentences)
- [ ] No conflicts between rules
- [ ] Does not contain domain-specific knowledge (use a skill)
- [ ] Does not define a persona (use an agent)

### Path-specific (`.instructions.md`)

- [ ] File is in `.github/instructions/` with `.instructions.md` extension
- [ ] Has `applyTo` glob in frontmatter
- [ ] Glob pattern is correct and tested
- [ ] Rules only apply to the matched file types
- [ ] Does not conflict with repo-wide instructions

## Common mistakes

| Mistake | Why it's wrong | Fix |
|---------|---------------|-----|
| Long paragraphs of background context | Instructions should be rules, not stories | Trim to actionable rules |
| Domain-specific docs in instructions | Loads for every interaction, adds noise | Move to a skill |
| Conflicting rules across files | Copilot behavior becomes non-deterministic | Remove conflicts |
| Path-specific file without `applyTo` | Instructions won't be scoped correctly | Add `applyTo` glob |
| Wrong glob pattern | Instructions apply to wrong files | Test glob patterns |
| "You are a..." persona in instructions | Instructions are rules, not personas | Move to an agent |
| Putting a workflow/task in instructions | Instructions are always-on, not on-demand | Move to a prompt |
| Adding API reference docs | Too much noise for every interaction | Move to a skill |
