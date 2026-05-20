---
allowed-tools: Bash(gh:*), Bash(git:*)
description: Summarize the currently checked out pull request
---

# Pull Request Summary

You are summarizing the currently checked out GitHub Pull Request.

Use:

!`gh pr view`

and

!`gh pr diff`

to understand the PR.

Also inspect the local codebase as needed for context.

Your goal is NOT to perform a detailed review.

Your goal is to quickly explain:

- what changed
- why it matters
- where risk likely exists
- where reviewer attention should focus

## Output Format

### Overview

In 2–4 sentences:

- What this PR does
- Why it exists
- Main architectural or product intent

### Areas Changed

List the major files, domains, or systems touched.

Example:

- Authentication flow
- Upload API
- React onboarding UI
- CDK infrastructure
- Database schema

### Key Changes

Highlight the most important modifications.

Focus on:

- behavioral changes
- architectural decisions
- migrations
- API changes
- infra changes
- anything surprising

### Risk Assessment

Choose one:

- Low Risk
- Medium Risk
- High Risk

Explain why.

Call out things like:

- auth changes
- concurrency
- state management
- migrations
- infra/CDK changes
- backward compatibility
- performance concerns

### Review Focus Areas

Where should reviewer attention go?

Example:

- Verify token refresh logic handles expiration correctly
- Double-check DynamoDB migration safety
- Confirm upload retries avoid duplicate submissions
- Validate tests cover unhappy paths

### Testing Confidence

Choose one:

- High
- Medium
- Low

Explain what gives confidence or concern.

## Important Rules

- Be concise.
- Prefer signal over exhaustive detail.
- Do not nitpick.
- Do not perform a full code review.
- Help the reviewer decide where to spend time.
