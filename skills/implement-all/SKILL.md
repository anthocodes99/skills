---
name: implement-all
description: >
  User wants to implement all ticket from a single spec, in this conversation.
disable-model-invocation: true
meta:
  version: 1.0.1
---

User wants to implement all the tickets of a single spec. Follow these steps:

1. Propose opening a new worktree by suggesting the name of the worktree. Worktree name should cover the spec's intent.
2. After worktree is created (or if user wants it on main), for each of the task, spawn an a coder agent to do the work. Start the agent's prompt with the following:

```
Implement the work described by the user in the spec or tickets.

Use /tdd where possible, at pre-agreed seams.

Run typechecking regularly, single test files regularly, and the full test suite once at the end.

Commit your work to the current branch.
```

Then continue with your own prompt. Be sure to give /tdd skill to the subagent.

3. Once coder agent done, use /code-review to review the work.
4. If the review found out issues with the code, determine whether to raise to the user or continue. Consider raising if the issue wasn't handled during spec & planning phase.
5. Then repeat Step 1 to Step 4 on each ticket.
6. Once done, use /finishing-a-development-branch to close off the branch.
