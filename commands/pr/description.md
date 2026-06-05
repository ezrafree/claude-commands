---
allowed-tools: Bash(gh:*), Bash(git:*)
description: Generate a concise GitHub PR description
---

# Pull Request Description

Generate a descriptive but concise GitHub Pull Request description for the currently checked out branch.

Use local git context to analyze changes:

!`git diff main..HEAD`

!`git status --short`

!`git log --oneline -10`

## Output Rules

Return exactly 3 paragraphs.

Wrap the entire response in a single fenced text code block for easy copy/paste.

Do not use headings.

Do not use bullet points.

Do not include markdown formatting inside the content.

Do not mention implementation details unless they help explain the change.

Keep the tone clear, professional, and concise.

## Paragraph Structure

Paragraph 1:
Summarize what changed and the primary purpose of the PR.

Paragraph 2:
Describe the most important implementation details, affected areas, or notable behavior changes.

Paragraph 3:
Describe testing, validation, risk, or follow-up considerations.

## Important

Focus on what a reviewer needs to understand before reviewing the PR.

Do not exaggerate the impact.

Do not invent tests or validation that are not evident from the diff.
