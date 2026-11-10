---
allowed-tools: Bash(gh:*), Bash(git:*), Bash(~/.claude/bin/prune-review-worktrees.sh:*), Bash(~/.claude/bin/check-review-request.sh:*), Read(~/worktrees/**)
argument-hint: [branch]
description: Review a pull request and draft review comments. Pass a branch to review it in its own worktree; omit it to review the current branch.
---

# Pull Request Review

You are reviewing a GitHub Pull Request.

Requested branch (blank if none was given): $ARGUMENTS

## Review Request Check (do this first)

This checks whether I, or a GitHub team I'm on, have been requested to review this PR. A previously submitted review by me also counts, since GitHub removes you from the requested list once you've reviewed and re-reviews would otherwise be blocked.

!`~/.claude/bin/check-review-request.sh $ARGUMENTS`

- If the line above starts with `REVIEW_REQUEST_OK`, continue with the review. Mention the reason in one line of the summary (e.g. "Requested via team org/platform").
- If it starts with `REVIEW_REQUEST_MISSING` or `REVIEW_REQUEST_ERROR`, or anything else, **pause here and ask me whether to proceed anyway**. Repeat the status line in the question. For `REVIEW_REQUEST_MISSING`, also mention that I could request a review from me or my team on the PR and re-run the command instead. Until I answer, do not set up a worktree, do not run any git commands, and do not review the code, even though the metadata and diff below are present.
  - If I say to proceed, continue with the review as normal, and mention in one line of the summary that it ran without a review request, repeating the status line.
  - Otherwise, stop and output only a short message saying the review was skipped.

## Housekeeping

Before this review starts, review worktrees whose PRs are merged or closed are swept automatically. Normally this prints nothing. Ignore its output unless it reports an error, and don't mention the sweep in the review.

!`~/.claude/bin/prune-review-worktrees.sh`

## PR Metadata

For the requested branch, or for the currently checked out branch when none was given:

!`gh pr view $ARGUMENTS --json number,title,body,baseRefName,headRefName,headRefOid,url`

Repository name: !`gh repo view --json name --jq .name`

Use this to understand the PR's purpose, author-provided context, and any temporary changes that will be removed before merge.

## Where to Inspect Code

- **No branch requested:** inspect code in the current working directory, as-is. Do not create a worktree.
- **Branch requested:** first confirm the metadata's `headRefName` matches the requested branch (or, if a PR number or URL was passed instead, that it matches the PR's `number` or `url`). If it doesn't match, stop and report the mismatch. Otherwise, set up a worktree as described below and inspect code there. Do not check out anything in the current working directory.

### Worktree setup (only when a branch was requested)

The worktree path is `~/worktrees/<repository name>/<number>-<headRefName>`, with every `/` in the branch name replaced by `-`. For example, PR 1234 from `feature/login-fix` in `auth-service` becomes `~/worktrees/auth-service/1234-feature-login-fix`. The PR number is what keeps the path unique; the branch name is only there for readability. Write the path unquoted so `~` expands.

1. Fetch the PR head through its pull ref, which also works for PRs opened from forks:
   `git fetch origin pull/<number>/head`
2. Run `git worktree list` to check whether the worktree path already exists.
   - If it doesn't, create it at the PR's exact head commit:
     `git worktree add --detach <worktree path> <headRefOid>`
   - If it does (a re-review after new pushes), move it to the new head:
     `git -C <worktree path> checkout --detach <headRefOid>`
     If this fails because the worktree has local changes, stop and report it. Never discard them.
3. Confirm `git -C <worktree path> rev-parse HEAD` prints `headRefOid`.

If a git command fails on a lock file (another review running in parallel), wait a few seconds and retry it once.

The worktree uses a detached HEAD so it never collides with a branch checked out elsewhere and always matches the diff exactly.

For the rest of the review, the worktree is the codebase. Read files by their full path under it and run git commands with `git -C <worktree path>`. Don't `cd` into it, and don't run `gh` from it; the metadata and diff in this prompt already cover everything `gh` is needed for.

Leave the worktree in place when you're done; it is removed automatically by the sweep above once its PR is merged or closed. Include its path and the manual cleanup command at the end of your output, as described in "Closing Lines".

## Code Changes

!`gh pr diff $ARGUMENTS || gh pr view $ARGUMENTS --json additions,deletions,changedFiles`

If the block above is a diff, use it to understand the code changes. If it's instead a JSON object with `additions`, `deletions` and `changedFiles` (GitHub refuses PR diffs over 20,000 lines, deleted lines included), build the diff locally as described in "Fallback: local diff" below. Either way, also inspect the codebase (the current working directory or the worktree, per the section above) as needed to understand surrounding context.

### Fallback: local diff (only when the diff above is unavailable)

GitHub's PR diff is a three-dot diff against the merge base, so a local diff from the merge base to `headRefOid` reproduces its hunks and line numbers. Run these with `git -C <worktree path>` in worktree mode, or plain `git` otherwise. They all name commits explicitly, so they never touch the working tree.

1. Fetch the PR head and base branch: `git fetch origin pull/<number>/head <baseRefName>`.
2. `git merge-base origin/<baseRefName> <headRefOid>` gives you `<merge base>`. If it differs from `git rev-parse origin/<baseRefName>`, the branch is behind its base. That's fine, since the diff still shows only the PR's changes, but mention it in the summary.
3. Confirm `git diff --shortstat <merge base> <headRefOid>` matches the JSON above. If it doesn't, say so in the summary and make every review comment general, because the line numbers can't be trusted.
4. Read everything except deleted files. Start with `git diff --numstat --diff-filter=d <merge base> <headRefOid>`, then read with `git diff --no-color --diff-filter=d <merge base> <headRefOid> -- <path>...` in batches small enough that the output isn't truncated (a few hundred lines each). Read source and config first. Skim or skip generated, vendored, and lock files, and list anything you didn't read in full in the summary.
5. Handle deleted files by listing them, not by reading their contents:
   - `git diff --dirstat=files,10 --diff-filter=D <merge base> <headRefOid>` shows which directories the deletions are in. `git diff --name-only --diff-filter=D <merge base> <headRefOid> [-- <dir>]` lists them.
   - Look for references to deleted files that remain in the PR head: `git grep -n -I -F -e <term> [-e <term>...] <headRefOid> --`. For terms, use the most specific directories that were removed entirely (check with `git ls-tree -d <headRefOid> -- <dir>`, and skip parents that still have files) plus the file names of deleted files outside those directories. Skip generic names like `README.md` or `index.md`. A match that still points at a deleted path is a finding (doc links, nav/index files, build or CI config, imports, CODEOWNERS). Matches that point at other repos or at surviving paths aren't.

Perform a thorough review with the following structure:

## 1. Summary

- State the PR number, branch, and short head commit SHA reviewed.
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

Call out things that are genuinely good, not just correct: a non-obvious edge case handled, a simplification that removes real complexity, a test that would actually catch a regression, a risky change rolled out safely, a PR description that made review easier. Be specific about what was done and why it matters.

Don't praise things that are merely expected, like following existing patterns, adding a test, or passing CI. If nothing stands out, say "Nothing notable" instead of inventing something.

Mark each note **worth posting** or **local only**. A note is worth posting when the author would learn something from hearing it, or when it reinforces a practice the team should keep. Mark at most one per review as worth posting, picking the strongest. It's fine, and often expected, to mark none when nothing clears that bar.

## 7. Verdict

Choose exactly one:

- Approve
- Request Changes
- Comment Only

Include a brief justification.

## 8. Review Comments (Ready to Post)

Include this section when the verdict is **Request Changes** or **Comment Only**, or when the verdict is **Approve** and section 6 has a note marked worth posting. Otherwise omit it.

Translate the findings from sections 3–5 into comments I can paste directly into the GitHub review UI. Every issue from section 3 must produce at least one comment. Code quality and testing findings produce comments only when they were worth mentioning above. The section 6 note marked worth posting, if there is one, produces a single praise comment. If none is marked, there is no praise comment.

### Determining placement

For each comment, decide whether it is **line-anchored** or **general**:

- **Line-anchored** — the feedback concerns a specific line or contiguous range of lines that appear in the diff. Anchor it there.
- **General** — the feedback spans multiple files, concerns something missing from the PR (e.g. absent tests, missing migration), or applies to the PR as a whole. Post it as a top-level review comment.

### Deriving line numbers

Line numbers must come from the diff, never from memory or the local working tree:

- Compute them from the `@@ -old,count +new,count @@` hunk headers in the diff: the `gh pr diff` output above or, in fallback mode, the local `git diff <merge base> <headRefOid>` output.
- In fallback mode, anchor only to lines in diff output you actually read. Deleted files weren't read line by line, so feedback about them goes in a general comment.
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

Order comments by severity (critical, then high, then code quality, then testing, then praise). Number them so they can be cross-referenced with sections 3 and 6.

### Writing the comments

- Write each comment to the PR author, in a direct, collegial tone. It must stand alone without the rest of this review.
- Lead with the problem and its impact, then the recommended fix. Keep it tight.
- Use GitHub-flavored markdown.
- Use inline code blocks where appropriate.
- For line-anchored comments where the fix is a small, unambiguous code change, include a GitHub `suggestion` block containing the exact replacement for the anchored line(s). Only do this when the replacement is complete and correct on its own; do not use `suggestion` blocks in general comments, as GitHub does not support them there.
- Prefix critical issues with **Blocking:** when the verdict is Request Changes.
- Do not restate the issue analysis from section 3 verbatim; rewrite it as feedback.

### Writing praise comments

- Anchor praise to the line or range it's about when there is one. Praise about the PR as a whole is a general comment, and on an Approve it can serve as the approval body.
- Say what specifically is good and why it matters, in one or two sentences. No generic "Great job!" or "LGTM!", no stacked exclamation points, no emojis.
- Never put praise in the same comment as an issue to soften it. Keep them separate so the issue reads clearly and the praise doesn't read as a cushion.
- Don't label it. No `praise:` prefix or any other tag like "Nice:" or "Kudos:". The wording itself should make it obvious it's appreciation and that no action is needed (e.g. "Good call handling the empty-list case up front, that's the input most likely to slip past the old check.").

## Closing Lines

End every completed review with these lines, after all other sections, in this order:

1. **Worktree mode only:** the worktree path and the manual cleanup command: `git worktree remove <worktree path>`.
2. The PR URL, taken from the metadata's `url` field, on its own line:
   `PR: <url>`

The PR URL is always the very last line of the output. Use the `url` field verbatim; never construct it from the repository name and number.

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
