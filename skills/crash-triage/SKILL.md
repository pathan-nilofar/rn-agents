---
name: "crash-triage"
description: "Takes a raw crash report or stack trace from a React Native app — Crashlytics, Sentry, a Play Console ANR, or a pasted red screen — and turns it into a triaged bug ticket: which layer it came from, the likely component, what to check first, and a reproduction path. Use when handed a stack trace, a crash report, a Crashlytics or Sentry issue, an ANR, or when asked to triage or investigate a crash."
---

# Crash Triage

Turn a stack trace into a ticket someone can actually pick up.

A raw crash report is mostly noise — framework frames, minified names, a device
model nobody needs. The useful signal is three or four lines, and finding them is
a skill that takes years. This encodes it.

## First: which layer crashed

React Native crashes come from three places, and the fix lives in a different part
of the codebase for each. Identify this before anything else.

| Signal in the trace | Layer | Where the fix lives |
|---|---|---|
| `TypeError`, `undefined is not an object`, `Cannot read property` | **JavaScript** | your React code |
| `RCTFatal`, `__exceptionPreprocess`, `EXC_BAD_ACCESS`, Objective-C frames | **iOS native** | a native module or its bridge |
| `java.lang.*`, `com.facebook.react.*`, `NullPointerException` | **Android native** | a native module or its bridge |
| `ANR`, `Input dispatching timed out`, main thread blocked | **Main-thread block** | work that should not be on the UI thread |
| `OutOfMemoryError`, `Jetsam`, memory pressure | **Memory** | image loading, list virtualisation, a leak |

## Then work through this

**1. Find the first frame that belongs to the app.** Skip framework and node_modules
frames. That line is where triage starts, not the top of the stack.

**2. Say what actually happened**, in one sentence, without jargon. "The feed screen
tried to read `.id` on a post that was undefined after a refresh."

**3. Name the likely cause.** Draw on the patterns that cause most React Native
crashes in practice:
- State updated after unmount — a fetch or timer resolving into a dead component
- A stale closure holding a value the render has since replaced
- Null from an API path that the type says cannot be null
- A native module called before it finished initialising
- An unhandled promise rejection that leaves state half-written
- Deep link or push notification opening a screen with no params

**4. Give the reproduction path** you would try first, as concrete steps. If the trace
does not contain enough to reproduce, say what is missing.

**5. Assign severity, and justify it:**

| | |
|---|---|
| **P0** | Crashes on launch, or in payments/auth. Blocks the release. |
| **P1** | Crashes a main flow. Fix before the next release. |
| **P2** | Edge case, recoverable, or needs an unusual sequence. |
| **P3** | Cosmetic, or a warning that never reaches a user. |

## What to produce

```markdown
**Layer:** JavaScript / iOS native / Android native / ANR / Memory
**Severity:** P0–P3, and why

**What happened**
<one sentence, no jargon>

**Where**
<file:line if the trace gives it, else the most likely component and why>

**Likely cause**
<the mechanism, not just the symptom>

**Check first**
1. <the cheapest thing that would confirm or eliminate this>
2. <the next one>

**To reproduce**
<steps, or exactly what information is missing>

**Suggested fix**
<the smallest change that addresses the cause, not the symptom>
```

## Getting it wrong

- **Say when you are unsure.** "The trace is minified and I cannot identify the
  component" is a useful answer. A confident wrong component sends someone down a
  day-long dead end.
- **Ask for the source map** when the trace is minified, rather than guessing at
  mangled names.
- **Do not suggest a try/catch as the fix** unless the cause genuinely is an expected
  failure. Swallowing an exception hides the crash; it does not fix it.
- If several crashes are pasted at once, group them by root cause before triaging —
  twenty reports are often one bug.
