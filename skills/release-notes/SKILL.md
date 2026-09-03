---
name: "release-notes"
description: "Turns merged pull requests since the last release tag into App Store and Play Store release notes written for users, not developers. Produces a store-ready block under 4000 characters, a longer internal changelog, and flags anything that needs a privacy-label or permissions review before submission. Use when preparing a mobile release, writing store release notes, drafting a changelog, or when asked what changed since the last version."
---

# Release Notes

Turn a list of merged pull requests into release notes a user will actually read.

Mobile teams write these by hand every sprint, late, and it shows. The store text
ends up as a bulleted list of ticket titles — "Fix FEED-2291", "Bump RN to 0.74" —
which tells a user nothing and tells the reviewer nothing either.

## What to do

**1. Gather the changes.** Run these; do not ask the user to paste anything:

```
git describe --tags --abbrev=0            # last release tag
git log <tag>..HEAD --merges --format='%s%n%b'
git log <tag>..HEAD --format='%s' --no-merges
```

If there is no tag, use the last 30 commits and say so.

**2. Sort every change into one of four buckets.** Discard anything that lands in
the fourth.

| Bucket | What belongs there |
|---|---|
| **New** | Something the user can now do that they could not before |
| **Improved** | Something that already existed and now works better or faster |
| **Fixed** | Something that was broken and now is not |
| **Invisible** | Refactors, dependency bumps, CI, tests, internal renames |

**3. Rewrite each surviving line from the user's side.** The test: could someone
who has never seen the codebase tell what changed for them?

> `feat(feed): implement cursor-based pagination for post list`
> → *"The feed now loads faster and keeps its place when you scroll back."*

> `fix: null guard in PaymentSheet.tsx onConfirm handler`
> → *"Fixed a crash that could happen when confirming a payment."*

Rules for the store block:
- Plain sentences. No ticket IDs, no file names, no library names, no version numbers.
- Lead with the change users will notice most, not the biggest diff.
- Never say "various bug fixes and improvements" — it is the sentence users skip.
- **Under 4000 characters** (Apple's limit). Play allows 500 per language, so also
  produce a short variant if the change list is long.

**4. Flag anything that needs a human before submission.** Do not guess — list it:
- A new permission, entitlement, or `Info.plist` / `AndroidManifest` key
- A new third-party SDK, which may change the privacy label
- Anything touching payments, authentication, or personal data
- A minimum-OS bump

## What to produce

```markdown
## Store release notes — v<version>
<the user-facing block, under 4000 characters>

## Short version — Play Store
<under 500 characters>

## Internal changelog
<grouped by New / Improved / Fixed, with PR numbers, for the team>

## Needs review before submission
<permissions, SDKs, privacy-label changes — or "nothing flagged">
```

## Getting it wrong

- **Do not invent a change that is not in the log.** If the log is thin, the notes are
  short. A short honest release note is fine; an invented feature is not.
- **Do not translate.** Ask which locales are needed and produce English first.
- **Do not guess the version number.** Read it from `package.json`, `build.gradle` or
  `Info.plist`, or ask.
- If a commit message is too vague to classify, list it under "could not classify" and
  ask, rather than inventing a user benefit for it.
