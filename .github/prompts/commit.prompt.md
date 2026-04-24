---
description: Review staged changes, optionally stage the intended files, update relevant docs, and create a conventional git commit for the current work.
agent: 'agent'
tools: ["read", "edit", "execute"]
---

Create a git commit for the current changes.

## Goal

Prepare a clean, well-explained commit that reflects the intended scope of work.

## Steps

1. Run `git diff --staged` to inspect what is already staged.
2. If nothing is staged, identify the relevant files for the requested change and stage only those files. Never use `git add .`.
3. Review whether `README.md` or nearby documentation should be updated to reflect the change. Update docs only when the change actually affects usage or setup.
4. Write a conventional commit message (`feat:`, `fix:`, `docs:`, `refactor:`, etc.).
5. Make the message explain why the change exists, not just what changed.
6. Create the commit with:

```bash
git commit -m "type(scope): subject" -m "body" -m "footer"
```

## Output

Report:

- Which files were included in the commit
- The final commit message
- Whether any documentation was updated

## Rules

- Never use `git add .`
- Do not include unrelated files just because they are modified
- Keep the subject line concise and imperative
- Prefer a body that explains motivation, impact, or constraints
