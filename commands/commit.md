---
allowed-tools: Bash(git:*)
description: Stage and commit changes
argument-hint: [commit-message]
disable-model-invocation: true
---

# Commit Changes

Create a git commit.

## Context

!`git status --short`

!`git diff HEAD`

!`git ls-files --others --exclude-standard`

## Rules

Commit message must:

- follow scoped conventional commit format
- be 50 characters or less
- be lowercase

Examples:

- `feat(navbar): add mobile menu`
- `fix(auth): prevent token loop`
- `refactor(api): simplify uploads`

## Behavior

If `$ARGUMENTS` is provided:

1. Validate the message:
   - scoped conventional commit format
   - 50 characters or less
2. If invalid:
   - explain why
   - suggest a corrected version
   - do not commit
3. If valid, run exactly once:

   `git add -v . && git commit -m "$ARGUMENTS"`

If `$ARGUMENTS` is empty:

1. Review the diff above
2. Generate a commit message that follows the rules
3. Run exactly once:

   `git add -v . && git commit -m "<generated-message>"`

Do not push.
