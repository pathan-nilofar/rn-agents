# rn-agents

Six agents for React Native work, defined entirely in markdown. No code.

```
skills/
  store-rejection/SKILL.md   App Store / Play rejection → what they actually meant
  rn-upgrade/SKILL.md        package.json → an ordered upgrade plan
  crash-triage/SKILL.md      stack trace → triaged ticket
  rn-perf-audit/SKILL.md     component → performance findings, by user impact
  release-notes/SKILL.md     merged PRs → store release notes
agents/
  rn-reviewer.md             a subagent that reviews a diff
```

The two newest are the ones that took the longest to write, because they are the ones
carrying knowledge that only comes from shipping:

**store-rejection** — Apple tells you which rule you broke and almost never tells you
what you did. *"Guideline 2.1 — Performance"* narrows it to about forty possible causes.
This decodes the common guideline numbers into what actually triggers them, ranks the
likely cause, and drafts the reply — including when to reply instead of resubmitting,
which is the part that costs teams a week.

**rn-upgrade** — reads `package.json`, finds which dependencies will break, and orders
the migration so that **every step leaves the app shippable**. Upgrades get abandoned
because someone attempts four versions in one branch and cannot ship for a month.

Each file is the whole agent. There is nothing else — no dependencies, no runtime of
mine, no build. You copy the file into a directory and the agent exists.

## Install

**Claude Code**

```bash
git clone https://github.com/pathan-nilofar/rn-agents
cp -r rn-agents/skills/*  ~/.claude/skills/     # or .claude/skills/ in a project
cp    rn-agents/agents/*  ~/.claude/agents/
```

Then just say what you want. The description in the frontmatter is what makes the
right agent load:

```
our app got rejected: Guideline 2.1 — Performance: App Completeness
plan an upgrade from RN 0.68 to 0.74
triage this crash: <paste stack trace>
why does the feed screen feel slow?
draft the release notes for this version
review my branch
```

**Cursor** — paste a `SKILL.md` body into `.cursorrules`, or save it under `.cursor/rules/`.

**Copilot** — paste the body into `.github/copilot-instructions.md`.

The frontmatter is Claude Code's format; the body is portable to anything that takes
a system prompt.

## Why markdown is a reasonable way to build an agent

I also wrote [rn-review](https://github.com/pathan-nilofar/rn-review), which is 700
lines of Python. These four are markdown. The difference is not effort — it is what
the task needs.

**Code, when the task is deterministic.** rn-review parses a diff, matches ten
patterns, posts to the GitHub API. Same input, same output, every time, free, in
milliseconds. Handing that to a model would make it slower, cost money per run, and
occasionally produce a different answer to the same diff.

**Markdown, when the task is judgement.** Turning `fix: null guard in PaymentSheet`
into *"Fixed a crash that could happen when confirming a payment"* is not
pattern-matching. Neither is reading a stack trace and saying which component is at
fault. There is no regex for those. What they need is written-down expertise, and a
model to apply it.

So the split in this repo is the same one inside rn-review: **anything expressible as
a rule should be a rule; the model is for what is left.** Getting that boundary right
is most of the work in building agents that are actually useful.

## What is in each file

Not "you are a helpful assistant." Each one carries the thing that takes years to
learn:

- **store-rejection** maps each Apple guideline number to what actually triggers it —
  2.1 is usually a missing demo account or an IPv6-only backend, 4.2 is a web wrapper,
  5.1.1 is a permission string that says what rather than why. None of that is in
  Apple's documentation; it comes from submissions.
- **crash-triage** opens with a table mapping trace signatures to which layer crashed
  — JavaScript, iOS native, Android native, ANR, memory — because the fix lives in a
  different part of the codebase for each, and identifying that first saves an hour.
- **rn-perf-audit** starts from what the user *feels* (a list that stutters, a tap that
  lags) and traces back to the cause, rather than listing tips.
- **release-notes** sorts every change into New / Improved / Fixed / Invisible and
  throws the fourth away, because a store note listing dependency bumps is a note
  nobody reads.
- **rn-reviewer** requires every comment to carry the cost of the mistake, not just
  its name, and is told explicitly that *"this looks good, nothing to flag"* is a
  complete review.

Every file also has a **"Getting it wrong"** section — what the agent should refuse to
guess at, and when to ask instead. An agent that confidently invents a component name
from a minified stack trace sends someone down a day-long dead end. Being specific
about failure modes is the difference between something a team keeps and something
they turn off in week two.

## Writing your own

The pattern is small enough to copy:

```markdown
---
name: "kebab-case-name"
description: "What it does and when to use it. This is what the runtime matches
  against, so write it as the situations someone would be in — not as a feature list."
---

# Title

<what problem this exists for, and why the obvious approach falls short>

## What to do
<the actual procedure — commands to run, decisions to make, in order>

## What to produce
<the exact output shape, so it is consistent every time>

## Getting it wrong
<what to refuse to guess at, and what to ask for instead>
```

The `description` matters more than it looks — it is the only thing the runtime sees
when deciding whether to load the agent. Write it as the situations someone would be
in, not as a summary of the file.

---

Built by [Nilofar Pathan](https://pathan-nilofar.github.io) — senior React Native
developer, six years across iOS and Android.
