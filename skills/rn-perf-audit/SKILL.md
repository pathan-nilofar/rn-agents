---
name: "rn-perf-audit"
description: "Audits a React Native screen or component for the render, list and bundle problems that make an app feel slow — unnecessary re-renders, unvirtualised lists, oversized images, blocking work on the JS thread, and imports that bloat the bundle. Produces findings ordered by user-visible impact with the fix for each. Use when a screen feels slow, a list stutters, startup time is bad, when profiling React Native performance, or when asked to review a component for performance."
---

# React Native Performance Audit

Find what is making a screen feel slow, in the order a user would notice.

Most performance advice is a list of tips. This works the other way round: start
from what the user feels, and trace it back to the code responsible.

## What the user feels, and what causes it

| Symptom | Usual cause |
|---|---|
| List stutters while scrolling | `ScrollView` instead of `FlatList`, no `getItemLayout`, rows re-rendering |
| Tap feels delayed | Work on the JS thread between the press and the state update |
| Screen flashes blank on open | Data fetched on mount with nothing rendered underneath |
| App slow to start | Everything imported at the top level, no lazy screens |
| Scrolling heats the phone | Images decoded at full size, unmemoised rows, animations on the JS thread |
| Typing lags in a form | Every keystroke re-rendering a parent |

## What to check, in this order

**1. Re-renders — the biggest and most common win**

- Object and array literals passed as props: `config={{ a: 1 }}` is a new reference
  every render, so `React.memo` on the child does nothing.
- Arrow functions in props: `onPress={() => go(id)}` has the same problem. In a list
  row it defeats memoisation for every visible row at once.
- Context holding a value that changes often — every consumer re-renders, however
  deep.
- State that lives higher than it needs to. A form field's state in a screen-level
  component re-renders the whole screen on every keystroke.

**2. Lists**

- `ScrollView` with `.map` mounts every child up front. Any list that can grow needs
  `FlatList`.
- `keyExtractor` returning the index means React reuses the wrong row on reorder.
- Missing `getItemLayout` when rows are a fixed height — it lets the list skip
  measurement.
- `renderItem` defined inline is a new function each render.
- Rows not wrapped in `React.memo`.

**3. Images**

- No explicit `width`/`height`, so layout shifts and the image decodes at full size.
- Full-resolution remote images rendered into a 60pt thumbnail.
- No caching strategy on a list that scrolls back and forth.

**4. The JS thread**

- `JSON.parse` or a large `.map`/`.filter` chain inside render.
- Animations driven from JS instead of `useNativeDriver: true` or Reanimated worklets.
- Synchronous storage reads during a navigation transition.

**5. Bundle and startup**

- Screens all imported at the top level instead of lazily by the navigator.
- A whole library imported for one function (`import _ from 'lodash'`).
- Dev-only code that ships because it is not behind `__DEV__`.

## What to produce

Order findings by **what the user would notice**, not by how easy they are to fix.

```markdown
## <Component or screen>

### 1. <finding> — <expected user-visible effect>
**Where:** file:line
**Why it costs:** <the mechanism>
**Fix:**
```diff
- <the current line>
+ <the replacement>
```

### 2. ...

## Measure it
<what to profile before and after, so the change is provable and not a guess>
```

## Getting it wrong

- **Do not claim a percentage.** "Roughly 40% faster" without a measurement is a
  guess wearing a number. Say what to measure instead.
- **Do not suggest memoising everything.** `useMemo` on a cheap value costs more than
  it saves and makes the code harder to read. Memoise what is expensive or what feeds
  a memoised child.
- **Check the assumption first.** If the list has eight items, `ScrollView` is fine
  and `FlatList` is the wrong advice.
- If the code is not visible, ask for the component rather than auditing in the
  abstract.

## Measuring

- **Flipper / React DevTools Profiler** — which components re-render, and why
- **`console.time`** around a suspected block, as the cheapest first check
- **Xcode Instruments** and **Android Studio Profiler** for native and memory
- **Hermes sampling profiler** for JS-thread work

Always measure before and after. A performance change without a number before it is
an opinion.
