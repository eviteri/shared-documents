---
name: copilot-customization-guide
description: Reference guide for choosing between GitHub Copilot instructions, skills, prompts, and agents. Use this when evaluating or creating customization files.
---

# Copilot Customization Guide

## Types Overview

### Instructions

- **Locations:**
  - `.github/copilot-instructions.md` (repo-wide)
  - `.github/instructions/**/*.instructions.md` (path-specific)
  - `AGENTS.md`, `CLAUDE.md`, `GEMINI.md` (repo root — agent instructions)
  - `$HOME/.copilot/copilot-instructions.md` (personal)
  - Directories in `COPILOT_CUSTOM_INSTRUCTIONS_DIRS` env var
- **Format:** Markdown (no frontmatter required for repo-wide, `.instructions.md` uses frontmatter with `applyTo` glob)
- **Always active:** Yes — loaded into every Copilot interaction
- **Purpose:** Broad, universal guidance that applies to all interactions in the repo

**Good fit when:**

- The rule applies to every Copilot interaction regardless of task
- It defines coding style, naming conventions, or language preferences
- It sets project-wide constraints ("always use TypeScript", "prefer functional components")
- It's short and doesn't need tools or special behavior

**Bad fit when:**

- It only applies to a specific domain or technology
- It requires tool access (fetch, shell, file write)
- It's a task template the user runs on demand
- It would add noise to unrelated interactions

**Examples:**

- "Use camelCase for variables and PascalCase for components"
- "Always write tests for new functions"
- "Prefer async/await over .then() chains"

### Skills

- **Locations (project):** `.github/skills/`, `.claude/skills/`, `.agents/skills/`
- **Locations (personal):** `~/.copilot/skills/`, `~/.claude/skills/`, `~/.agents/skills/`
- **Filename:** must be exactly `SKILL.md` in its own subdirectory
- **Format:** Markdown with required YAML frontmatter (`name`, `description`)
- **Always active:** No — Copilot loads it when the description matches the current context
- **Purpose:** Specialized instructions, scripts, and resources for a specific domain or recurring task

**Required frontmatter:**

```yaml
---
name: lowercase-with-hyphens
description: What the skill does and when Copilot should use it.
---
```

**Good fit when:**

- It's specialized guidance, examples, scripts, or reference material for a focused domain
- It should load automatically when the topic is relevant
- It may include additional resources in the skill directory that Copilot can use when relevant
- Multiple agents or prompts could benefit from this knowledge

**Bad fit when:**

- It needs its own frontmatter-based tool scope
- It should apply to every interaction (use instructions instead)
- The user should invoke it manually as a reusable prompt every time
- It needs its own persona or behavioral constraints (use an agent)

**Examples:**

- REST API patterns for a specific framework
- Database query conventions
- Error handling playbooks
- Internal API documentation

### Prompts

- **Location:** `.github/prompts/<prompt-name>.prompt.md` (project level)
- **Format:** Markdown with YAML frontmatter (commonly `description`, plus `agent` and `tools` when needed)
- **Always active:** No — user invokes manually
- **Purpose:** Reusable task templates for specific workflows

**Practical tip:** Start with the smallest frontmatter that works. In this repo, prompts generally use `description`, `agent: 'agent'`, and a minimal `tools` list.

**Good fit when:**

- It's a repeatable task the user triggers on demand
- It follows a defined sequence of steps
- It produces a specific output (a file, a report, a transformation)
- Different users would run it the same way

**Bad fit when:**

- It should load automatically based on context (use a skill)
- It defines an ongoing persona with constraints (use an agent)
- It's a universal rule (use instructions)
- It only contains reference docs with no actionable steps

**Examples:**

- "Initialize project schemas from an API"
- "Generate unit tests for a file"
- "Create a changelog from recent commits"
- "Scaffold a new component"

### Agents

- **Locations:** `.github/agents/` (project) or `~/.copilot/agents/` (personal)
- **Format:** Markdown with YAML frontmatter (always `description`, often `tools`, optionally `name`)
- **Always active:** No — user invokes with `@agent-name`
- **Purpose:** A specialized persona with its own tools, constraints, and behavior

**Practical tip:** Prefer conservative agent frontmatter unless a GitHub doc explicitly justifies extra fields like `model` or `target`.

**Good fit when:**

- It needs its own identity and behavioral rules
- It requires tool access (fetch, read, write, shell)
- It has constraints that differ from default Copilot behavior (read-only, specific API access)
- It combines skills, tools, and judgment into a coherent role

**Bad fit when:**

- It's just reference docs with no actions or constraints (use a skill)
- It's a one-off task with defined steps (use a prompt)
- The rules should apply globally (use instructions)
- It doesn't need tools — it's just knowledge

**Examples:**

- "A read-only API query agent"
- "A security reviewer that flags OWASP issues"
- "A documentation specialist"
- "A database migration assistant"

## Decision Flowchart

Ask these questions in order:

1. **Should it apply to EVERY interaction in this repo?** → Instructions
2. **Should Copilot auto-load specialized instructions/resources when relevant?** → Skill
3. **Is it a repeatable task the user runs on demand?** → Prompt
4. **Does it need its own persona, tools, or constraints?** → Agent

## Validation Checklist

### Instructions

- [ ] Applies universally to all interactions
- [ ] Does not require tool access
- [ ] Is concise and won't add noise to unrelated tasks
- [ ] Does not overlap or conflict with other instructions

### Skills

- [ ] Has required frontmatter (`name`, `description`)
- [ ] `name` is lowercase with hyphens
- [ ] `description` clearly states when Copilot should load it
- [ ] Matches the skill model: specialized instructions/resources/scripts loaded when relevant
- [ ] Does not declare tool access in frontmatter
- [ ] Skill directory name matches the `name` field

### Prompts

- [ ] Has frontmatter with `description`
- [ ] Defines a clear, repeatable task with steps
- [ ] Produces a specific output or outcome
- [ ] Tools listed in frontmatter are necessary for the task
- [ ] Uses the smallest frontmatter shape that satisfies the task
- [ ] Would not be better as auto-loaded knowledge (skill)

### Agents

- [ ] Has frontmatter with `description`
- [ ] Defines a clear persona with behavioral constraints
- [ ] Requires tool access that justifies being an agent
- [ ] Uses optional frontmatter only when clearly justified
- [ ] Constraints differ from default Copilot (e.g., read-only, specific API)
- [ ] Does not duplicate what instructions or skills already cover

## Common Misclassifications

| Symptom | Current type | Should be |
|---------|-------------|-----------|
| Has `tools` in frontmatter | Skill | Agent or Prompt |
| "Always do X" with no domain scope | Skill or Agent | Instructions |
| Domain-specific instructions/resources that should auto-load | Prompt or Agent | Skill |
| Manually-invoked task with defined output | Skill or Agent | Prompt |
| Persona with constraints + tool access | Skill or Prompt | Agent |
| Only relevant in one domain, loaded every time | Instructions | Skill |
