---
allowed-tools: Bash(git:*)
description: Generate a scoped conventional commit message
argument-hint: [optional-context]
---

# Commit Message Generator

Generate a scoped conventional commit message for the current git changes.

Use:

!`git status --short`

!`git diff --stat`

!`git diff`

!`git ls-files --others --exclude-standard`

Additional context from user:

$ARGUMENTS

## Rules

The commit message must:

- be scoped conventional commit format
- be 50 characters or less total
- include the type, scope, parentheses, colon, and space in the 50-character limit
- be lowercase
- be imperative
- describe the change clearly
- avoid vague words like `update`, `changes`, `stuff`, or `misc`

Allowed types:

- feat
- fix
- refactor
- chore
- docs
- test
- style
- perf
- build
- ci

Format:

type(scope): message

Examples:

- `feat(nav): add mobile menu`
- `fix(auth): handle expired token`
- `refactor(api): simplify uploads`

## Output

Return only one commit message.

Do not explain.
Do not include markdown.
Do not commit.
