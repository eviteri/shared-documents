---
name: copilot-best-practices-agents
description: Best practices for creating GitHub Copilot custom agents (.agent.md files). Use this when reviewing, creating, or troubleshooting agent files.
---

# Copilot Agent Best Practices

## What is an agent?

An agent is a specialized Copilot persona with its own identity, tools, and behavioral constraints. Users invoke it with `@agent-name`.

## When to use an agent

- The task needs a **distinct persona** with specific expertise
- It requires **tool access** (fetch, read, write, shell)
- It has **constraints** that differ from default Copilot (e.g., read-only, specific API)
- It combines knowledge, tools, and judgment into a **coherent role**

## When NOT to use an agent

- It's just reference docs with no actions → use a **Skill**
- It's a one-off task with defined steps → use a **Prompt**
- The rules should apply globally to all interactions → use **Instructions**
- It doesn't need tools — it's just knowledge → use a **Skill**

## File location and naming

- **Location:** `.github/agents/` (project) or `~/.copilot/agents/` (personal)
- **Extension:** `.agent.md` (required)
- **Naming:** lowercase with hyphens (e.g., `security-reviewer.agent.md`)
- Home directory agents override repo agents with the same name

## Frontmatter fields

Start with the smallest schema that works. Add optional fields only when you have a clear reason.

### Required

```yaml
---
description: What this agent does and when to use it.
---
```

### Optional

```yaml
---
name: Agent Display Name
description: What this agent does and when to use it.
tools: ["read", "search", "edit"]
model: Claude Sonnet 4.6
target: vscode
disable-model-invocation: false
user-invocable: true
mcp-servers:
  server-name:
    url: https://example.com/mcp
---
```

| Field | Type | Default | Purpose |
|-------|------|---------|---------|
| `name` | string | filename | Optional display name |
| `description` | string | — | What the agent does and when to use it |
| `tools` | list | `["*"]` (all) | Tools the agent can access |
| `model` | string | inherited | LLM model to use |
| `target` | string | both | `vscode` or `github-copilot` |
| `disable-model-invocation` | boolean | false | Prevent auto-selection, require manual invoke |
| `user-invocable` | boolean | true | Whether users can manually select this agent |
| `mcp-servers` | object | — | Additional MCP servers and tools |
| `metadata` | object | — | Annotations |

For most project agents in this repo, `description` plus a minimal `tools` list is enough.

### Tools configuration

- `["*"]` or omitted → all tools enabled
- `[]` → no tools
- `["read", "search", "edit"]` → only specific tools
- Supports MCP namespacing: `"server-name/tool-name"`

## Body content (max 30,000 characters)

The Markdown body below the frontmatter defines behavior. It should include:

1. **Role definition** — who the agent is and what it does
2. **Behavioral constraints** — what it must and must not do
3. **Workflow steps** — how it should approach tasks
4. **Examples** — sample inputs and expected outputs (when helpful)

## Checklist

- [ ] File is in `.github/agents/` with `.agent.md` extension
- [ ] Filename is lowercase with hyphens
- [ ] Has `description` that explains purpose AND when to use it
- [ ] `tools` list only includes tools the agent actually needs
- [ ] Avoids optional frontmatter unless it solves a real problem
- [ ] Body defines a clear persona, not just reference docs
- [ ] Has explicit constraints (what NOT to do)
- [ ] Does not duplicate what instructions or skills already cover
- [ ] Body is under 30,000 characters

## Common mistakes

| Mistake | Why it's wrong | Fix |
|---------|---------------|-----|
| Agent with no tools and only docs | It's just a skill with extra steps | Move to `.github/skills/` as a SKILL.md |
| `tools: ["*"]` when it only needs read access | Overly permissive, security risk | List only needed tools |
| Copies every optional field from an example | Increases schema risk and maintenance overhead | Start with `description` and the smallest useful `tools` list |
| Description says "helps with everything" | Copilot can't decide when to use it | Be specific about the domain and trigger |
| Body has "always use semicolons" type rules | Universal rules belong in instructions | Move to `copilot-instructions.md` |
| Agent that runs a fixed sequence of steps | That's a prompt, not a persona | Move to `.github/prompts/` |
| No constraints section | Agent has no boundaries | Add explicit "do NOT" rules |
| Filename has spaces or uppercase | Won't work programmatically | Use lowercase-with-hyphens.agent.md |
