---
name: generate-uat
description: >
  Generates a flow-based UAT (User Acceptance Test) document for the UI module
  affected by a feature or bug fix. Tests the entire module's user journey, not
  just the changed code. Use whenever the user says "create UAT", "generate UAT",
  "write UAT", "UAT for this", "help me create UAT", or points to a spec file
  and asks for test cases. Always trigger when a spec path is mentioned alongside
  any UAT/testing intent, even if phrased casually.
---

# generate-uat

Generates a flow-based UAT document. The UAT tests the entire UI module touched by the
change by walking through the natural user journey as a single seamless test session.

**Before generating, confirm the content outline with the user.** Show the planned flows
and any column tables before writing the file. Do not save until the user approves.

## Inputs

The user provides a path to a Markdown spec file. Example trigger phrases:
- "create UAT for `docs/superpowers/specs/2026-06-30-audioplayers-uat.md`"
- "generate UAT based on this bug fix / feature" + spec path
- "can you help me create UAT based on this?" pointing at a spec

## Workflow

### Step 1 — Read the spec

Read the spec file the user provided. Extract:
- What changed (feature, bug fix, migration, refactor)
- Which module or screen is affected (if named)
- Any UI elements, screens, flows, or user actions referenced

### Step 2 — Explore the codebase

Search the project for screens, routes, widgets, or pages related to the
affected module. Goal: reconstruct the natural user journey through that part
of the app — entry points, key actions, branching scenarios, exit points.

Useful search strategies:
- Grep for screen/widget names mentioned in the spec
- Find route definitions or navigation files to understand screen flow
- Look for the main widget/screen file and read its key interactions

**For every dialog or form in the flow, read the form initialization code** to
confirm what each field starts as (empty, 0.00, pre-filled, etc.). Do not assume
from the component's render — find where the form state is seeded (e.g. `useEffect`,
`defaultValues`, initial state object). A dialog that looks like it would pre-fill
the balance often starts at 0.00; assert only what the code confirms.

**If a browser automation tool is available, walk the actual flow live before finalizing wording.** Code tells you a dialog exists; only clicking it live confirms the exact button/label text as currently rendered — copy can be edited after the code you read, badges get shortened, defaults get changed mid-session. Prefer the live-observed string over an inferred one whenever they'd differ. This matters most for anything a step tells the tester to read verbatim (dialog titles, button labels, badge text).

**Capture section names and their visible descriptions too, not just buttons.** A screen's real label for a group of controls (e.g. a card header reading "Services & products", with a helper line underneath it) is what the tester actually sees — write steps against that name, never a name you invented for it (don't call it "the rate grid" if the screen calls it "Services & products"). If the screen prints its own explanation of a control, prefer quoting that over writing your own description of what it does.

### Step 3 — Gather missing inputs

After exploring the codebase, collect any information you still need before drafting:

**Always ask if not already provided:**
- **App URL** — the staging or production URL the tester will open. Do not guess or invent a URL.

**Ask only if still unclear after reading the spec and codebase:**
- "Which screen does this change affect?"
- "Is there a specific flow beyond [inferred flow] you want covered?"

Do not ask upfront about flows or screens — investigate first. But always ask for the URL if missing.

### Step 4 — Identify flows

Choose the right anchor for the flows:
- **By module** (default for new features) — cover the full user journey through the
  affected UI module, not just the changed code.
- **By spec** (for amendments to existing features) — scope flows tightly to what
  the spec changed, assuming the rest of the module already has coverage.

Ask the user which approach they want if it is not clear from context.

Group the user journey into discrete named flows. Each flow covers one coherent
slice of the module (e.g. "Add audio track", "Playback controls", "Profile
switching"). Flows should be:

- **Sequential** — designed to run one after another as a single test session.
  Steps pick up where the previous flow left off where natural.
- **Non-repetitive** — don't navigate back to the same screen across flows
  unless testing a meaningfully different scenario.
- **Continue in place** — if the last action of a flow already lands the tester
  on the screen the next flow needs (e.g. "Duplicate" opens straight into the
  new draft), start the next flow there. Don't insert a "go back to the list
  first" step just for tidiness — that's a redundant round-trip the tester has
  to perform for no testing value.
- **Concrete** — every step names a specific UI element or action. Never say
  "click the button" — say "tap the Play button in the audio player bar".
- **Give every flow a short, stable code** (e.g. `F04-UAT-01`, `F04-UAT-02`, ...
  using the feature/ticket prefix if one exists) in both the Summary table and
  the flow's own heading (`## F04-UAT-01 — Scheme list & duplicate`). This is
  how the flow gets referenced later — in bug reports, in follow-up UATs that
  continue from it, in conversation.

**Two kinds of flows — write both, don't default to only the first:**

- **UI-verification flows** check that a screen behaves correctly on its own:
  a dialog opens with the right fields, a menu item is enabled/disabled in the
  right states, a value persists after save. These stay on one screen (or one
  screen + its dialogs).
- **End-to-end business-outcome flows** take an action on one screen, then go
  check a *different* screen or downstream system to see whether the correct
  result was actually produced — e.g. place and pay an order, then check a
  reporting page for the resulting computed figure. These catch bugs that
  UI-verification flows structurally cannot, because the failure lives in what
  happens *after* the button click looks successful, not in the click itself.
  When writing one:
  - Find the trigger action and trace where its result actually surfaces —
    it is often a different page/module than the one you acted on.
  - Pull the expected numbers from the **live** config/admin screen for that
    data immediately before writing the step, not from a spec document — specs
    drift from what's actually configured, especially in an actively-developed
    environment.
  - If the underlying engine is async/queue/stream-driven, say so in the step
    or Fail-if wording ("allow a short delay — accrual is asynchronous")
    instead of implying the result appears instantly.
  - Watch for **date-input parsing traps**: typing a date into a raw HTML date
    field can silently parse in the wrong day/month order and land the test
    data in the wrong period. Add a Fail-if for "result lands in the wrong
    period" whenever a step asks the tester to type a date.

### Step 5 — Draft the content outline

Before writing the file, show the user:
- The planned flows (names + one-line objectives)
- The proposed **Business Context** (see below)
- The proposed **Overview** and **Getting Started** text

Wait for the user to approve or correct before saving.

### Step 5.5 — Verify every "verify X shows Y" assertion from code

Before writing any step that tells the tester to verify a displayed value (a badge,
a status, a calculated total), trace where that value is derived in the code and
confirm it. Do not infer from partial reading — follow the derivation to its source.

Common traps:
- A badge color/label that looks obvious may be driven by a multi-condition function
  with a non-obvious priority order (e.g. financial status derived from netPaid,
  totalAmount, depositAmount in a specific sequence).
- A value that "should" reflect the refunded amount may not — check what the
  recalculation function actually computes and saves.

If the derivation is ambiguous or depends on runtime data you can't confirm statically,
note it as "verify" rather than asserting a specific value.

**In a shared/actively-developed dev environment, values drift mid-session.** A rate,
badge label, or scheme you captured earlier in this same conversation may no longer
match — from your own test edits, or someone else's, or a hot-reload. Re-check any
concrete number immediately before writing it into a step; don't reuse an
earlier-session snapshot on trust.

### Step 6 — Generate the UAT document

Read `references/uat-template.md` for the exact output format. Key formatting rules:

- **Sub-steps** (3.1, 3.2…) must be indented with exactly **3 spaces** per line.
- **Tables** inside a step must be **flush (no indent)** — indented tables break the Markdown renderer.
- **Result callouts** after sub-step groups must be labeled with the range, e.g. `Result for 3.1 to 3.3`.
- **Precondition** must use plain UI-facing language — no internal permission strings or code names.

**Write for someone with zero knowledge of the system.** The tester may not be technical and doesn't know the app's internal vocabulary. Concretely:
- Reference on-screen labels **verbatim** — section headers, column names, badge text. Never invent a name for something the UI already names (the screen says "Services & products" → the step says "Services & products", not "the rate grid" or "the configurator table").
- Describe a result as **what the eye sees**: the exact text and how it looks (color, badge style, icon) — not what happened internally ("the box now shows the text **Tiered** with a blue background", not "the cell's tiered flag is set").
- Don't explain interaction mechanics the tester doesn't need. Say what to click, not how the UI reveals it ("click the `⋯` next to it", not "hover to reveal the `⋯` icon, then click it").
- Avoid dev/internal terms outright — swap them for the plain equivalent: "cell" → "box", "grid" → the section's real on-screen name, an internal field/enum name (e.g. `practitioner_class`) → a one-clause plain-language description of the business concept behind it.
- This applies to Business Context and Overview too, not just Steps — no internal field names, enum values, or system-architecture language a non-technical reader wouldn't know.

**Business Context** section — derive from the spec and codebase, or ask the user:
- **Who uses it:** the user role and their goal in one sentence.
- **Key rules:** business constraints a tester needs to judge correctness (e.g. data scoped to logged-in user, exports expire, only confirmed orders count). List as bullets if more than one.
- **Source of truth:** how the tester verifies a result is correct (e.g. cross-check against another page or report).

If any of these cannot be confidently derived, ask the user before writing the file.

Then generate the UAT document and save it to:

```
docs/uat/YYMMDD-<topic>-uat.md
```

Where `<topic>` matches the spec file's topic slug. Example: if the spec is
`2026-06-30-audioplayers-migration-design.md`, save to
`docs/uat/260630-audioplayers-migration-uat.md`.

Create the `docs/uat/` directory if it does not exist. After saving, tell the
user the full file path.
