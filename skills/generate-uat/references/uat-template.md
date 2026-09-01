# UAT document format

Load this file when generating the UAT document. Use the structure below exactly.

## Full document structure

```markdown
# UAT — [Module / Feature Name]

> Generated from: `path/to/spec.md`

## Overview

[2–3 sentences: what this module does, who uses it, and what was built or changed. Use plain language — avoid technical jargon. End with one sentence stating what this UAT is validating.]

## Business Context

**Who uses it:** [The user role and what they are trying to accomplish — one sentence.]

**Key rules:** [Business constraints a tester needs to know to judge whether a result is correct. E.g. data is scoped to the logged-in user only, exports expire after X hours, only confirmed orders count toward totals. List as short bullets if more than one.]

**Source of truth:** [How the tester knows if a result is right — e.g. "compare the Total Sales figure against the Orders page filtered by the same staff and date range".]

## Getting Started

[How to reach the module from a fresh browser session. Always include the app URL. Keep it to 2–3 steps.]

## Summary

| Code | Flow | Status |
|------|------|--------|
| [PREFIX-UAT-01] | [Flow 1 name] | TBD |
| [PREFIX-UAT-02] | [Flow 2 name] | TBD |
| [PREFIX-UAT-03] | [Flow 3 name] | TBD |

---

## [PREFIX-UAT-01] — [Flow 1 name]

**Objective:** [One sentence — what this flow validates]

**Precondition:** [App state, permissions, or data required before starting this flow]

**Steps:**
1. [Action — name specific UI element]
2. [Action] → verify [expected result inline]
3. [Action with branching sub-tests]

   3.1 [Sub-test A label] — [what to check?]
   3.2 [Sub-test B label] — [what to check?]
   3.3 [Sub-test C label] — [what to check?]

> [!TIP] Result for 3.1 to 3.3
> - [ ] Success
> - [ ] Failed

4. [Continue main flow after sub-tests]
5. [Action] → verify [expected result]

**Fail if:**
1. [Named failure condition — specific, not vague]
2. [Another named failure condition]

> [!TIP] Result
> - [ ] Success
> - [ ] Failed

**Remark:**

---

## [PREFIX-UAT-02] — [Flow 2 name]

[Same structure as above]
```

## Rules for applying this template

**Summary table:** Always at the top. One row per flow. Status always starts as `TBD`. Do not mark any status in the generated document — the tester fills this in.

**Flow codes:** Every flow gets a short, stable code (`PREFIX-UAT-01`, `PREFIX-UAT-02`, ...) in both the Summary table and its own `##` heading. Pick `PREFIX` from the feature/ticket id if one exists (e.g. `F04` for a spec titled "PH2-F-04"); otherwise use a short slug of the module name. Numbers stay stable once assigned — if a flow is added later mid-document, append it at the next number rather than renumbering existing ones.

**Flows:** Write as many flows as needed to cover the module. Minimum 2, no hard maximum. Each flow has a clear distinct objective.

**Inline verification:** For steps with a single clear expected outcome, embed the check inline: `5. Tap play → verify audio starts`. Do not create a sub-step for this.

**Sub-steps:** Use sub-steps (3.1, 3.2...) only when a single action branches into multiple distinct pass/fail checks — e.g. testing multiple options in a settings panel. Each sub-step group gets its own `Result` callout immediately after the sub-steps.

**Sub-step formatting rules:**
- Sub-steps must be indented with exactly **3 spaces** per line.
- Any table that belongs to a parent step must be **flush (no indent)** — indented tables break the renderer.

**Result callout placement:**
- At the end of every flow (always)
- After every sub-step group (when sub-steps are present), labeled with the range: `Result for 3.1 to 3.3`
- NOT after every individual step

**Fail if:** List failure conditions that are specific and named. Not "something goes wrong" but "audio does not resume after returning from profile 2".

**Remark:** Leave blank. The tester fills this in.

**Precondition:** State the minimum app state needed to start this flow. Use plain language — write permission names as the user sees them in the UI, not as internal permission strings (e.g. "Report Access permission", not "`report_read`"). Do not mention data setup requirements that are already seeded for testing. If a flow naturally continues from the previous one (e.g. the user is already on a screen from flow 1), say so: "Continuing from [previous flow name] — user is on the [screen name] screen."

## Voice

Write every step as if the tester has never seen the app and doesn't know its internal
vocabulary. The Example below is the tone reference — match its level of hand-holding,
not the shorter/technical phrasing you might default to. In particular:

- Name on-screen labels **verbatim** (section headers, column names, badge text) —
  never invent a name for something the screen already names.
- Describe results as **what the eye sees** (exact text + its visual treatment —
  color, badge shape, icon) rather than what happened internally.
- Say what to click, not how the UI reveals it — skip mechanics like "hover to reveal".
- Swap dev/internal terms for plain equivalents before they reach the document
  ("cell" → "box", an internal field/enum name → a plain-language description of
  the business concept it represents).

## Example

```markdown
## NOTIF-UAT-02 — Quiet hours

**Objective:** Check that turning on Quiet Hours actually silences notifications during the set window, and that the toggle's state is visible correctly.

**Precondition:** Continuing from "Add a notification rule" — the tester is on the Notifications settings screen, with at least one rule already created.

**Steps:**
1. On the **Notifications** screen, find the section titled **Quiet Hours**. Underneath it there's a short line of grey text explaining what it does, and a toggle switch on the right.
2. Click the toggle. It should turn from grey to blue, and slide to the right.
3. Two time boxes should now appear underneath, labeled **Starts** and **Ends**, each showing a default time.
4. Click the **Starts** box and change it to a time 2 minutes from now. Click **Save** at the bottom of the section.
5. Wait until that time passes, then trigger a test notification (use the **Send test notification** button under Developer Options).

   5.1 During the Quiet Hours window — the notification should not appear on screen, and the bell icon in the top bar should show a small "zzz" badge instead of the usual red dot.
   5.2 After the Quiet Hours window ends — send another test notification and verify it appears normally, with the usual red dot on the bell icon.

> [!TIP] Result for 5.1 to 5.2
> - [ ] Success
> - [ ] Failed

6. Click the Quiet Hours toggle again to turn it off. It should turn back to grey and slide left, and the Starts/Ends boxes should disappear.

**Fail if:**
1. The toggle's color/position doesn't match its on/off state.
2. A notification still appears on screen during the Quiet Hours window.
3. The bell icon doesn't show the "zzz" badge while Quiet Hours is active.

> [!TIP] Result
> - [ ] Success
> - [ ] Failed

**Remark:**
```
