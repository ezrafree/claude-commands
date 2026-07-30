---
description: Draft my daily standup (DSU) in my speakable first-person format
argument-hint: [optional extra items to include, e.g. "mention the pilot prep call"]
---

# Draft my DSU

Write my daily standup update. I read this out loud in the meeting, so it must sound like natural first-person speech — complete sentences I can say in one breath — not a written status report.

## Gather evidence

1. Best source: work done in this conversation, if any. Use it first — it is the freshest and most accurate.
2. Then fill gaps from GitHub and git. Run these (skip any that fail):

```bash
gh search prs --involves @me --updated ">=$(date -v-3d +%F)" --limit 20
```

```bash
git log --author="$(git config user.email)" --since="3 days ago" --oneline
```

Filter the results to the previous business day (on Mondays, that is Friday). Distinguish what I authored/drove from what I merely commented on.

3. Fold in anything I passed as arguments: $ARGUMENTS
4. If after all that "Yesterday" would be empty or guesswork, ask me one short question about what I worked on — never invent items.

## Format

Output exactly this shape, inside one fenced markdown block, with nothing after it:

```markdown
# DSU

## Yesterday

- <top-level accomplishment, past tense>
  - <supporting detail, one breath long>
  - <supporting detail>
- <next accomplishment>

## Today

- I'll <first plan>
- and I'll <second plan>
```

## Style rules

- First person, contractions, spoken register: "I fixed", "I verified", "I'll work to get", "and I'll continue chasing".
- 2-4 top-level bullets per section; nested sub-bullets carry the detail. Every line must be speakable in one breath.
- Reference PRs as `repo#123` in backticks (e.g. `auth0#1960`). Never raw URLs — I can't read a URL out loud.
- Fold blockers into "Today" as an action I'm taking ("I'll continue chasing an approval from identity on ..."). Only add a `## Blockers` section if something is truly stuck with no action available to me.
- Lead with outcomes; drop process noise (reruns, tooling detours, dead ends) unless it changed the outcome or the team needs to know.
- No emojis, no jargon I wouldn't say out loud.

## Example of my voice

```markdown
# DSU

## Yesterday

- I identified and fixed the root cause of the grow-dev deploy failures that SSS-459 introduced
  - There were three stacked causes I found:
    - the deploy CLI can't grant scopes to its own client
    - the manual grant had gone to the User-Delegated Access tab, which doesn't apply to M2M tokens
    - and the scope list was missing a couple of scopes which I've now added
  - I verified end-to-end and the import is green, so Guild Employer SSO is now live in the dev environment
  - I documented the whole grant procedure so we'll be ready for staging and prod rollouts
  - I also reworked PR `auth0#1960` to re-enable the dev import now that it's working

## Today

- I'll work to get `auth0#1960` approved and merged
- and I'll continue chasing an approval from identity on `auth0-actions#506`
```
