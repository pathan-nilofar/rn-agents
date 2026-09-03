---
name: "store-rejection"
description: "Decodes an App Store or Google Play rejection into what the reviewer actually meant, what in the app triggered it, what to change, and a draft reply to App Review. Covers Apple guideline numbers (2.1, 4.2, 5.1.1, 3.1.1 and the rest) and Play policy strikes. Use when an app submission is rejected, when a reviewer's message is unclear, when a binary is stuck in review, when a Play update is blocked, or when preparing a resubmission or an appeal."
---

# Store Rejection Decoder

Apple tells you which rule you broke. It almost never tells you what you did.

A rejection reads *"Guideline 2.1 — Performance: App Completeness"* and the team spends
two days guessing. The guideline number narrows it to roughly forty possible causes;
the reviewer's one-line note narrows it to three. Getting from there to the actual fix
is experience, and this is that experience written down.

## First, extract these from the rejection

Ask for whatever is missing rather than guessing:

- **The guideline number** — the single most useful token in the message
- **The reviewer's note** — the free-text line, which is where the real cause hides
- **Any attached screenshot** — usually shows the exact screen that failed
- **Device and OS** they tested on — many rejections are one-device-specific
- **Submission number** and whether this is a first submission or an update
- **Whether the same binary passed before** — if yes, look at what changed

Then say plainly: **what they mean**, **what probably triggered it**, **what to change**,
and **whether to resubmit or reply first**.

## Apple — what each guideline actually means in practice

### 2.1 · App Completeness
By far the most common rejection, and the vaguest. The real cause is almost always one
of these, in rough order of likelihood:

- **No demo account, or the one supplied does not work.** Anything behind a login must
  have working credentials in App Review Notes. Reviewers do not sign up.
- **Crash on launch on their device**, often an older one, or on first launch with no
  cached data.
- **A feature that needs hardware they do not have** — camera, NFC, a paired device —
  with no fallback and no explanation.
- **Placeholder content** — lorem ipsum, "Coming soon", a broken image, a dead link.
- **Backend not reachable** from their network. Staging URLs left in a release build
  cause this constantly.
- **IPv6-only network failure.** Apple reviews on IPv6-only. A hardcoded IPv4 literal
  or an IPv4-only backend fails there and nowhere else.

**Ask first:** does the build have working demo credentials, and does it point at
production?

### 2.3 · Accurate Metadata
Screenshots showing a feature the build does not have, a description promising more
than it delivers, or screenshots from a different device class. Also fires when the
app name or subtitle contains a competitor's name.

### 2.5.1 · Software Requirements
Private API use. In React Native this is usually a third-party SDK, not your code.
Ask which native dependencies were added since the last accepted build.

### 3.1.1 · In-App Purchase
Selling digital goods or services without IAP, or linking out to an external payment
page. This is the rejection that costs the most to fix because it is a business-model
problem, not a code one. Physical goods and real-world services are exempt — digital
content is not.

### 3.1.2 · Subscriptions
Missing subscription terms, no restore-purchases path, unclear renewal wording, or
the price and period not shown before purchase.

### 4.2 · Minimum Functionality
*"Your app is primarily a web view."* Fires on thin wrappers around a website. The fix
is native functionality that could not exist in a browser — push notifications, offline
support, camera, biometrics, widgets.

### 4.3 · Spam
Duplicate of another app on the store, often a white-label build. Hard to argue.
Differentiate the app, or consolidate.

### 4.8 · Login Services
If the app offers any third-party login (Google, Facebook, Apple ID excepted), it must
also offer **Sign in with Apple**, or an equivalent that limits data collection.

### 5.1.1 · Data Collection and Storage
Several distinct causes under one number:
- **Permission strings missing or unhelpful.** Every `NS*UsageDescription` must say why
  in plain language. "This app needs camera access" is rejected; "Take a photo to add
  to your profile" passes.
- **Asking for data the app does not need**, or asking at launch rather than at the
  point of use.
- **No account deletion.** If the app creates an account, it must let the user delete
  it *from inside the app*. A support-email link is not enough.
- **Privacy policy link missing or dead.**

### 5.1.2 · Data Use and Sharing
The **privacy nutrition label** does not match what the app actually collects. Adding
an analytics or crash SDK and not updating the label triggers this. Also **App Tracking
Transparency**: using IDFA without the ATT prompt.

### 5.1.5 · Location Services
Background location without a clear reason, or the location permission prompt not
explaining the benefit to the user.

### 1.2 · User-Generated Content
Any app where users can post content needs: a way to **report** content, a way to
**block** users, a published moderation policy, and a stated response window. Missing
any one of the four triggers this.

## Google Play — the common ones

| Reason | What it means |
|---|---|
| **Data safety form mismatch** | The declared form does not match what the SDKs collect. Check every dependency, not just your code. |
| **Target API level** | Play enforces a minimum `targetSdkVersion` that rises each year. Purely a build-config fix. |
| **Background location** | Requires a separate declaration and a demo video showing why. Most apps should drop the permission instead. |
| **Permissions declaration** | `QUERY_ALL_PACKAGES`, SMS and Call Log need a specific approved use case. |
| **Deceptive behaviour** | Functionality that does not match the listing, or an undisclosed ad SDK. |
| **Families policy** | Anything reachable by under-13s pulls in a much stricter ruleset. |

Play strikes are **account-level**, not app-level. Repeated strikes risk the developer
account, so treat a Play rejection as more serious than an Apple one.

## What to produce

```markdown
## Rejection — Guideline <n> · <name>

**What they mean**
<plain language, one short paragraph>

**Most likely trigger**
<ranked, with the reason each is likely given their note>

**What to check, in order**
1. <cheapest check that would confirm or eliminate the top cause>
2. ...

**What to change**
<concrete — file, setting, or the flow to add. Config where it is config.>

**Resubmit or reply?**
<see below>

**Draft reply to App Review**
<only when replying is the right move>
```

## Resubmit, or reply first

**Reply first** when the reviewer has misunderstood something, when the feature works
but they could not reach it, or when the fix is a clarification rather than a change.
A reply is answered in hours; a resubmission restarts the queue.

**Resubmit** when something is genuinely broken or missing.

**Never do both at once** — a new binary while a reply is open resets the thread and
loses the reviewer's context.

## Writing the reply

Reviewers are people working through a queue. What works:

- **Answer the specific point.** Do not restate the app's value.
- **Say what changed**, in one sentence, with the build number.
- **Give exact steps** to reach the feature they could not find, including credentials.
- **Never argue the guideline.** Arguing that the rule should not apply loses. Showing
  that it does not apply, factually, wins.
- **Keep it under 200 words.**

If the same rejection comes back twice with no new information, request a call through
App Review, or escalate through the Resolution Center. Do not resubmit a third time
unchanged.

## Getting it wrong

- **Do not guess the guideline number** if it is not in the message. Ask for the full
  rejection text — half of these are misdiagnosed because someone paraphrased it.
- **Do not assume it is a code problem.** A large share of rejections are metadata,
  permission strings, or App Review Notes — nothing to do with the binary.
- **Do not promise a timeline.** Review times vary and an expedited request is not
  granted for a self-inflicted rejection.
- **Guidelines change.** Apple revises the App Review Guidelines several times a year
  and Play policy more often. Confirm anything version-specific against
  `developer.apple.com/app-store/review/guidelines/` before acting on it.
- If the rejection mentions **legal, IP, or a named rights-holder**, say plainly that
  it needs a lawyer rather than drafting an argument.
