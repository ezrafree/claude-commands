---
allowed-tools: Bash(gh:*), Bash(date:*), Bash(/bin/date:*), mcp__claude_ai_Google_Calendar__list_events, mcp__claude_ai_Google_Calendar__list_calendars, mcp__claude_ai_Google_Calendar__search_events
description: Generate a Slack-ready daily standup summary
argument-hint: [optional-context]
---

# Daily Standup

Generate a Slack-ready standup update summarizing the last working day: my own PR activity, PRs I reviewed, and calendar meetings.

The window is yesterday. If today is Monday, the window is Friday through Sunday.

Use:

!`/bin/date "+Today: %A, %Y-%m-%d"`

!`/bin/date -v-$(( $(/bin/date +%u) == 1 ? 3 : 1 ))d "+Window start (last working day): %Y-%m-%d"`

!`gh api user --jq '"GitHub login: " + .login'`

My PRs created, updated, or merged since the window start:

!`gh search prs --author=@me --updated=">=$(/bin/date -v-$(( $(/bin/date +%u) == 1 ? 3 : 1 ))d +%Y-%m-%d)" --limit 50 --json repository,number,title,url,state,isDraft,createdAt,updatedAt,closedAt --jq '[.[] | {repo: .repository.nameWithOwner, n: .number, state, draft: .isDraft, title, created: .createdAt, updated: .updatedAt, closed: .closedAt, url}]'`

Candidate PRs I may have reviewed (search cannot filter by review date, so these MUST be verified):

!`gh search prs --reviewed-by=@me --updated=">=$(/bin/date -v-$(( $(/bin/date +%u) == 1 ? 3 : 1 ))d +%Y-%m-%d)" --limit 50 --json repository,number,title,url,state,updatedAt --jq '[.[] | {repo: .repository.nameWithOwner, n: .number, state, title, updated: .updatedAt, url}]' -- -author:@me`

Additional context from user:

$ARGUMENTS

## Steps

### 1. My PRs

For each PR in the first search result, classify its verb using the timestamps:

- merged: state is merged and closed falls within the window
- opened: created falls within the window
- updated: everything else

Timestamps are UTC and the window is a local date, so compare leniently. Drop PRs whose only activity clearly predates the window.

For each PR that remains, fetch its details so you can describe what it actually does:

- `gh pr view <n> --repo <owner/repo> --json title,body,files,additions,deletions,state,mergedAt`
- Only if the body and file list are not enough to understand the change: `gh pr diff <n> --repo <owner/repo>`

Write one sentence (two only if truly necessary) describing the actual change and its purpose. Do not restate the title.

### 2. Reviews

For each candidate in the second search result, verify I actually reviewed it during the window:

- `gh api repos/<owner>/<repo>/pulls/<n>/reviews --jq '[.[] | {user: .user.login, state, submitted_at}]'`

Keep only PRs with a review by my GitHub login whose submitted_at is on or after the window start. Drop all others. Drop any PR I authored.

Record the review outcome: approved, commented, or requested changes. Fetch PR details as in step 1 to write the one-sentence description.

### 3. Meetings

If the Google Calendar tools (mcp__claude_ai_Google_Calendar__list_events) are available in this session, list events for the window on the primary calendar.

Include only real meetings. Exclude all-day events, declined events, focus time, and out-of-office blocks.

Each meeting becomes a short bullet with its title, plus a few words of context only when useful.

If the Calendar tools are not available, omit the Meetings section from the update and add one line after the code block noting that calendar data was unavailable.

### 4. Other

If additional context from the user was provided above, summarize it as bullets in an Other section.

## Output Rules

Wrap the entire update in a single fenced text code block for easy copy/paste into Slack.

Use Slack formatting inside the block: *bold* section labels and • bullets. No markdown headings, no dashes as bullets.

First line: `*Standup — <window>*` where window is like `Thu Jul 23` or `Fri Jul 18 – Sun Jul 20`.

Sections in order, each label on its own line: *My PRs*, *Reviews*, *Meetings*, *Other*.

Omit any section with no items. Do not write "none".

PR bullets: `• repo#number (status) — description.` Status is one of merged, open, draft, or closed for My PRs; approved, commented, or requested changes for Reviews. Use the short repo name; include the owner only if two repos in the update share a name.

One sentence per PR. Describe what the change does, not what the title says.

Keep the whole update scannable: aim for about 10 lines inside the block.

Do not invent PRs, reviews, or meetings. Only include items confirmed by the data above.

Do not add commentary outside the code block, except the single calendar-unavailable note when it applies.

Example:

```text
*Standup — Thu Jul 23*

*My PRs*
• webapp#412 (merged) — Adds retry with exponential backoff to the upload worker so transient S3 failures no longer drop files.
• api#98 (open) — Moves tenant resolution from a per-request DB lookup into the JWT claims to cut auth latency.

*Reviews*
• infra#77 (approved) — Terraform change moving Redis into a private subnet with new security group rules.

*Meetings*
• Sprint planning
• 1:1 with Dana

*Other*
• Debugged the staging deploy pipeline
```
