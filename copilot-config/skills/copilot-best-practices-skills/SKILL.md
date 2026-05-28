---
name: copilot-best-practices-skills
description: Best practices for creating GitHub Copilot agent skills (SKILL.md files). Use this when reviewing, creating, or troubleshooting skill files.
---

# Copilot Skill Best Practices

## What is a skill?

A skill is a folder of instructions, scripts, and resources that Copilot loads automatically when it detects the topic is relevant.

## When to use a skill

- It's **reference material** (API docs, framework patterns, debugging guides)
- It provides **specialized instructions or resources** for a focused domain task
- It should load **automatically** when the topic comes up
- It may include **scripts, examples, or supplementary docs** in the skill directory
- Multiple agents or prompts could benefit from this knowledge

## When NOT to use a skill

- It needs its own direct tool scope in frontmatter → use an **Agent**
- It should apply to every interaction → use **Instructions**
- It should be manually invoked on demand every time → use a **Prompt**
- It needs a persona with constraints → use an **Agent**

## File location and naming

- **Location:** `.github/skills/<skill-name>/SKILL.md`
- **Filename:** must be exactly `SKILL.md` (case-sensitive)
- **Directory name:** lowercase with hyphens (e.g., `strapi-v5/`)
- Directory name should match the `name` field in frontmatter
- Can include additional files (scripts, examples, docs) in the same directory

### All valid locations

**Project-level (repo-specific):**

| Path |
|------|
| `.github/skills/<name>/SKILL.md` |
| `.claude/skills/<name>/SKILL.md` |
| `.agents/skills/<name>/SKILL.md` |

**Personal (shared across projects):**

| Path |
|------|
| `~/.copilot/skills/<name>/SKILL.md` |
| `~/.claude/skills/<name>/SKILL.md` |
| `~/.agents/skills/<name>/SKILL.md` |

## Frontmatter fields

### Required

```yaml
---
name: skill-name-lowercase
description: What this skill provides and when Copilot should load it.
---
```

### Optional

```yaml
---
name: skill-name-lowercase
description: What this skill provides and when Copilot should load it.
license: MIT
allowed-tools: shell
---
```

| Field | Type | Purpose |
|-------|------|---------|
| `name` | string | Unique identifier — lowercase, hyphens for spaces |
| `description` | string | What the skill does AND when Copilot should use it |
| `license` | string | Applicable license |
| `allowed-tools` | string | Pre-approved tools (use with caution) |

### Writing a good description

The `description` is critical — Copilot uses it to decide when to load the skill. It must:

- State **what** the skill provides
- State **when** to use it (trigger context)
- Be specific enough to avoid false matches
- Be broad enough to catch relevant queries

**Good:**
```yaml
description: Strapi v5 REST API patterns using flattened documentId format. Use this when working with Strapi v5 projects or APIs.
```

**Bad:**
```yaml
description: Strapi stuff
```

## Body content

The Markdown body should contain structured instructions and supporting material:

- Clear headings and sections
- Examples, scripts, or referenced resources when useful
- Guidance on when and how Copilot should use the skill
- Enough domain detail to help Copilot perform the task well when the skill auto-loads

## Security warning

> Only pre-approve `shell` or `bash` in `allowed-tools` if you have reviewed every script in the skill directory and fully trust their source. Pre-approval removes confirmation prompts and risks arbitrary command execution.

## Checklist

- [ ] File is named exactly `SKILL.md` in its own directory
- [ ] Directory name is lowercase with hyphens
- [ ] Has `name` in frontmatter (lowercase, hyphens)
- [ ] Has `description` that explains purpose AND when to load
- [ ] `name` matches the directory name
- [ ] Matches GitHub's current skill model: instructions/resources/scripts that auto-load when relevant
- [ ] Does not list `tools` in frontmatter (skills don't get tools — that's for agents)
- [ ] `allowed-tools` only used if scripts are reviewed and trusted
- [ ] No persona or behavioral constraints (that's an agent)

## Common mistakes

| Mistake | Why it's wrong | Fix |
|---------|---------------|-----|
| File not named `SKILL.md` | Copilot won't find it | Rename to exactly `SKILL.md` |
| Missing `name` or `description` | Copilot can't index or auto-load it | Add required frontmatter |
| `name` has uppercase or spaces | Breaks naming convention | Use lowercase-with-hyphens |
| Directory name doesn't match `name` | Confusing, may cause issues | Align them |
| Description is vague ("helps with code") | Copilot loads it for everything or nothing | Be specific about domain and trigger |
| Skill has `tools` in frontmatter | Skills don't declare tool access in frontmatter | Move to an agent if it needs direct tool scope |
| Should be manually invoked every time for one task | That's a prompt, not an auto-loaded skill | Move to `.github/prompts/` |
| Contains "You are a..." persona | That's an agent definition | Move to `.github/agents/` |
| `allowed-tools: shell` without review | Security risk — enables arbitrary commands | Remove or review all scripts first |
