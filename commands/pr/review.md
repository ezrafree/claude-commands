---
allowed-tools: Bash(gh:*), Bash(git:*)
description: Review the currently checked out pull request
---

# Pull Request Review

You are reviewing the currently checked out GitHub Pull Request.

Use the output of:

!`gh pr diff`

to understand the changes.

Also inspect the local codebase as needed to understand surrounding context.

Perform a thorough review with the following structure:

## 1. Summary

- Briefly describe what this PR does.
- Identify the main components or areas affected.

## 2. Key Changes

- Highlight the most important modifications.
- Call out risky or complex changes.

## 3. Issues & Concerns

Only include issues that are:

- critical
- high

Omit:

- medium issues
- low issues
- nitpicks
- minor style concerns

Look for:

- bugs or likely bugs
- logic errors
- missing edge cases
- security concerns
- performance problems
- risky architectural choices
- regressions

For each issue include:

### Priority
Critical / High

### Location
File, function, or area affected

### Problem
What is wrong

### Why It Matters
Real-world impact

### Suggested Fix
Concrete recommendation

## 4. Code Quality

Only mention meaningful maintainability concerns.

Cover:

- readability
- naming consistency
- function and variable design
- comments/documentation when important

## 5. Testing

Assess whether tests sufficiently cover:

- happy paths
- edge cases
- regressions

Call out missing tests only when meaningful.

## 6. Positive Notes

Explicitly call out good decisions, thoughtful implementation details, or especially clean code.

## 7. Verdict

Choose exactly one:

- Approve
- Request Changes
- Comment Only

Include a brief justification.

## Important Rules

- Be specific and reference relevant parts of the diff.
- Do not invent line numbers.
- Do not comment on unchanged code unless directly affected.
- Favor signal over volume.
- If no major concerns exist, say so clearly.
