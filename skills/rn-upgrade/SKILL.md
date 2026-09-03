---
name: "rn-upgrade"
description: "Plans a React Native version upgrade. Reads package.json, finds which dependencies will break, orders the migration into steps that each leave the app shippable, and flags the native-side changes that are the real work. Use when upgrading React Native, when stuck on an old version, when a dependency demands a newer RN, when planning a New Architecture migration, or when asked how hard an upgrade will be."
---

# React Native Upgrade Planner

Turn "we should upgrade some day" into an ordered plan someone can start on Monday.

Teams postpone RN upgrades for years, then attempt four minor versions in one branch,
break the app, and abandon it. The upgrade is rarely hard because of React Native
itself — it is hard because of the native side and because of dependencies that were
last published two years ago.

## Read the ground truth first

Run these; do not ask the user to paste anything:

```
cat package.json
cat ios/Podfile 2>/dev/null | head -30
cat android/build.gradle 2>/dev/null | head -40
node -v && ruby -v 2>/dev/null && java -version 2>&1 | head -1
ls ios/*.xcworkspace android/app/src/main/AndroidManifest.xml 2>/dev/null
git log -1 --format=%cd                    # how stale is this branch
```

Establish: **current RN version**, **target version**, **which native dependencies
exist**, and **whether the New Architecture is already on**.

## The version cliffs

Most versions are routine. These are the ones that cost real time — when a plan crosses
one of these, it becomes its own step:

| Crossing | Why it hurts |
|---|---|
| **→ 0.60** | Autolinking replaces manual linking; AndroidX migration; CocoaPods becomes required |
| **→ 0.68** | New Architecture becomes available (opt-in). Do not turn it on during a version jump. |
| **→ 0.69** | React 18 — `StrictMode` double-invokes effects in dev and surfaces existing bugs |
| **→ 0.70** | Hermes becomes the default engine; JSC-specific behaviour and stack traces change |
| **→ 0.71** | Flexbox `gap` support; TypeScript template becomes the default |
| **→ 0.73** | Kotlin version bump on Android; new debugger |
| **→ 0.74+** | Yoga 3 layout changes; minimum iOS version rises |
| **New Arch on** | Every native module must support TurboModules/Fabric. The largest single step. |

**Version specifics move fast. Confirm every one of these against the changelog and the
Upgrade Helper before acting on it — do not rely on this table alone.**

## The two tools that do most of the work

- **`react-native-upgrade-helper`** (`react-native-community.github.io/upgrade-helper/`)
  — a file-by-file diff between any two versions. This is the authoritative source for
  what changes in the native projects.
- **`npx react-native upgrade`** — applies that diff. It works well on a clean project
  and badly on one with heavily customised native folders. Check `git status` after.

## Triage the dependencies — this is where upgrades actually die

For every native dependency in `package.json`:

1. **When was it last published?** Anything untouched for over 18 months is a risk.
2. **Does it support the target RN version?** Check its peer dependencies and issues.
3. **Does it support the New Architecture**, if that is the goal?
4. **Is it still the right choice?** Some have been replaced by a core API or a
   maintained fork.

Sort every one into:

| | |
|---|---|
| **Fine** | Recent, supports the target, no action |
| **Bump** | Needs a version bump, no code change |
| **Migrate** | API changed — code work, scoped |
| **Replace** | Unmaintained or incompatible. **This is the real cost of the upgrade.** |

Call out the *replace* list early and loudly. A team that discovers halfway through
that its payment SDK or its VoIP module has no compatible version has already lost the
sprint.

## Shape the plan

**Rule: every step must leave the app shippable.** A four-week branch that cannot be
released is how upgrades get abandoned when a priority lands.

A plan that works:

1. **Prepare** — upgrade JS-only dependencies, fix warnings, get the test suite green
   on the *current* version. Ship it.
2. **Bump one minor version.** Apply the Upgrade Helper diff, resolve native conflicts,
   test both platforms, ship it.
3. **Repeat** — one version per step. Two at once means an unresolvable bisect when
   something breaks.
4. **Replace incompatible dependencies** as their own steps, before the version that
   needs them.
5. **New Architecture last**, and only as a deliberate project — never bundled with a
   version bump.

## What to produce

```markdown
## Upgrade: RN <current> → <target>

**Verdict:** <a sentence: is this routine, or does it need a dedicated project?>
**Cliffs crossed:** <the ones from the table, and why each matters here>

### Dependencies
| Package | Current | Status | Action |
|---|---|---|---|
<one row each — Fine / Bump / Migrate / Replace>

**Blockers:** <anything in Replace, and what it costs>

### The plan
<numbered steps, each shippable, with what to verify before moving on>

### Native work
**iOS:** <Podfile, deployment target, Xcode version, entitlements>
**Android:** <Gradle, Kotlin, compileSdk, AGP>

### Test before each step ships
<the flows most likely to break given what changed>

### Rough effort
<a range, with what would make it the high end>
```

## Getting it wrong

- **Do not give a single number for effort.** "Two days" is a guess wearing a
  number. Give a range and say what widens it.
- **Do not skip versions** because the diff looks small. Version-by-version keeps a
  bisect meaningful.
- **Do not fold the New Architecture into a version bump.** When something breaks you
  will not know which change caused it.
- **Verify the version table.** React Native moves quickly and this file will age.
  The Upgrade Helper and the release changelog are authoritative; this is a map.
- **Do not plan around a native module you have not opened.** If a dependency is
  critical and its compatibility is unclear, say so and make checking it step zero.
- If the native folders are heavily customised, say plainly that `npx react-native
  upgrade` will conflict and the diff must be applied by hand.
