---
allowed-tools: Bash(jj st:*), Bash(jj commit:*), Bash(jj diff:*), Bash(jj describe:*), Bash(jj squash:*), Bash(jj new:*)
description: Create a jj commit
---

## Context

- Current jj status:
!`jj status`

- Current jj diff (working copy changes):
!`jj diff`

- Recent commits:
!`jj log -l 10`

## Your task

We use jj (Jujutsu) for version control.
Based on the above changes:

- Review the context and understand what changes are present
- If necessary, understand what should be added to .gitignore
- Create commits by grouping relevant changes using these AI-safe commands:
  - `jj commit -m 'description'` (commits all current changes)
  - `jj squash <filepaths> -m 'description'` (move specific files to parent)
  - `jj describe -m 'description'` (set description for current working copy)
  - `jj new <base>` (create new working copy commit)

## Important Notes for AI Agents

- **NEVER use interactive commands** like `jj split` without files, `jj squash -i`, or `jj resolve`
- **Always specify `-m` flag** to avoid interactive editor
- **Working copy is automatically committed** - changes are tracked immediately
- **Use file-specific operations** for selective commits: `jj squash <files>`
- **No staging area** - all changes in working copy are included unless specified otherwise
