---
allowed-tools: Bash(gh:*), Bash(git:*)
description: Review the currently checked out pull request and draft review comments
---

# Pull Request Review

You are reviewing the currently checked out GitHub Pull Request.

Use the output of:

!`gh pr view --json title,body,baseRefName,headRefName,url`

to understand the PR's purpose, author-provided context, and any temporary changes that will be removed before merge.

Use the output of:

!`gh pr diff`

to understand the code changes.

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

## 8. Review Comments (Ready to Post)

Include this section **only** when the verdict is **Request Changes** or **Comment Only**. If the verdict is Approve, omit it entirely.

Translate the findings from sections 3–5 into comments I can paste directly into the GitHub review UI. Every issue from section 3 must produce at least one comment. Code quality and testing findings produce comments only when they were worth mentioning above.

### Determining placement

For each comment, decide whether it is **line-anchored** or **general**:

- **Line-anchored** — the feedback concerns a specific line or contiguous range of lines that appear in the diff. Anchor it there.
- **General** — the feedback spans multiple files, concerns something missing from the PR (e.g. absent tests, missing migration), or applies to the PR as a whole. Post it as a top-level review comment.

### Deriving line numbers

Line numbers must come from the diff, never from memory or the local working tree:

- Compute them from the `@@ -old,count +new,count @@` hunk headers in `gh pr diff`.
- For added or unchanged lines, use the **new-file** (right-side) line number.
- For deleted lines, use the **old-file** (left-side) line number and mark the side as `LEFT`.
- Multi-line anchors must be a single contiguous range within one hunk.
- If you cannot determine the exact line with confidence, downgrade the comment to **general** and name the file and function in the body instead. Never guess.

### Output format

Precede each comment with a one-line placement header, then the comment body in a fenced `markdown` block. Use a four-backtick outer fence if the body itself contains a fenced code block.

For line-anchored comments:

**Line comment — `path/to/file.ext` line 42** (or `lines 42–47`; add `(LEFT)` for deleted lines)

For general comments:

**General PR comment**

Order comments by severity (critical, then high, then code quality, then testing). Number them so they can be cross-referenced with section 3.

### Writing the comments

- Write each comment to the PR author, in a direct, collegial tone. It must stand alone without the rest of this review.
- Lead with the problem and its impact, then the recommended fix. Keep it tight.
- Use GitHub-flavored markdown.
- Use inline code blocks where appropriate.
- For line-anchored comments where the fix is a small, unambiguous code change, include a GitHub `suggestion` block containing the exact replacement for the anchored line(s). Only do this when the replacement is complete and correct on its own; do not use `suggestion` blocks in general comments, as GitHub does not support them there.
- Prefix critical issues with **Blocking:** when the verdict is Request Changes.
- Do not restate the issue analysis from section 3 verbatim; rewrite it as feedback.

## Important Rules

- Be specific and reference relevant parts of the diff.
- Do not invent line numbers.
- Do not comment on unchanged code unless directly affected.
- Favor signal over volume.
- If no major concerns exist, say so clearly.

## Temporary PR Environments

Read the PR description before evaluating environment-related changes.

If the PR description says that a PR environment, preview environment, temporary infrastructure, or related configuration will be removed before merge:

- Treat those changes as temporary review context.
- Do not report their mere presence as an issue or concern.
- Do not choose "Request Changes" solely because the temporary environment is still present.
- Do not require its removal during review.
- You may mention it neutrally in the summary if it materially helps explain the PR.

Still report an environment-related issue when it independently creates a critical or high risk, such as:

- exposing secrets or sensitive data
- modifying production or shared infrastructure
- granting unsafe permissions or public access
- destructive operations
- substantial uncontrolled cost
- behavior that would remain after the temporary environment is removed

Do not assume an environment is temporary unless the PR description or diff makes that intent clear.
