---
name: copilot-best-practices-prompts
description: Best practices for creating GitHub Copilot prompt files (.prompt.md). Use this when reviewing, creating, or troubleshooting prompt files.
---

# Copilot Prompt Best Practices

## What is a prompt?

A prompt is a reusable task template that users invoke manually. It defines a specific workflow with defined steps and expected output. Users trigger it with `/prompt-name` in Copilot Chat.

## When to use a prompt

- It's a **repeatable task** the user triggers on demand
- It follows a **defined sequence of steps**
- It produces a **specific output** (a file, a report, a transformation)
- Different users would run it the same way

## When NOT to use a prompt

- It should load automatically based on context → use a **Skill**
- It defines an ongoing persona with constraints → use an **Agent**
- It's a universal rule → use **Instructions**
- It only contains reference docs with no actionable steps → use a **Skill**

## File location and naming

- **Location:** `.github/prompts/` (project level)
- **Extension:** `.prompt.md` (required)
- **Naming:** lowercase with hyphens (e.g., `generate-tests.prompt.md`)
- Available in VS Code, Visual Studio, and JetBrains IDEs
- Invoked with `/prompt-name` in Copilot Chat

## Frontmatter fields

Prefer the minimal prompt frontmatter first. Add more only if the latest GitHub docs clearly require it.

```yaml
---
description: What this prompt does.
agent: 'agent'
tools: ["read", "edit"]
---
```

| Field | Type | Purpose |
|-------|------|---------|
| `description` | string | What the prompt does |
| `agent` | string | Set to `'agent'` for multi-step prompts that need agent capabilities |
| `tools` | list | Tools the prompt can use when `agent: 'agent'` is set |

For most prompts here, the safest baseline is `description`, `agent: 'agent'`, and the smallest tool list that can complete the task.

## Variables

Prompts support dynamic input with this syntax:

```
${input:variableName:Display prompt text for the user}
```

Example:
```markdown
Explain the following code for a ${input:audience:Who is the target audience?}:

${input:code:Paste the code to explain}
```

## Body content

The body should define a clear task:

1. **Goal** — what the prompt achieves
2. **Steps** — ordered sequence of actions
3. **Output format** — what the result should look like
4. **Rules** — constraints on how to perform the task

## Checklist

- [ ] File is in `.github/prompts/` with `.prompt.md` extension
- [ ] Filename is lowercase with hyphens
- [ ] Has `description` in frontmatter
- [ ] Defines a clear, repeatable task with steps
- [ ] Produces a specific output or outcome
- [ ] `agent: 'agent'` is set if it needs tool access
- [ ] Tools listed are only those needed for the task
- [ ] Avoids extra frontmatter fields unless they are clearly needed
- [ ] Would not be better as auto-loaded knowledge (skill)
- [ ] Does not define a persona (that's an agent)

## Common mistakes

| Mistake | Why it's wrong | Fix |
|---------|---------------|-----|
| Prompt file in `.github/agents/` | Wrong directory — prompts go in `prompts/` | Move to `.github/prompts/` |
| No `agent: 'agent'` but uses tools | Tools won't be available for the prompt agent flow | Add `agent: 'agent'` |
| Copies optional fields from older examples without a need | More likely to trigger IDE schema warnings | Keep prompt frontmatter minimal |
| No clear steps or output | It's reference material, not a task | Move to a skill |
| "You are a..." persona definition | That's an agent, not a task template | Move to `.github/agents/` |
| Lists tools it doesn't actually use | Unnecessary permissions | Remove unused tools |
| Only contains knowledge/docs | Prompts are for tasks, not reference | Move to `.github/skills/` |
| No variables for user input | May not be reusable across contexts | Add `${input:...}` where appropriate |
| Filename has `.agent.md` extension | Copilot treats it as an agent | Rename to `.prompt.md` |
