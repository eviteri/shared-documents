---
name: commit
description: Create a well-formatted git commit
disable-model-invocation: true
---

Create a git commit for the current changes:

1. Run `git diff --staged` to see what's staged
2. Update the README.md file with any relevant information about the changes
3. If nothing is staged, stage the relevant files (never use `git add .`)
4. Write a conventional commit message (feat:, fix:, docs:, etc.)
5. Focus on WHY, not WHAT
6. Create the commit with `git commit -m "type(scope): subject" -m "body" -m "footer"`
