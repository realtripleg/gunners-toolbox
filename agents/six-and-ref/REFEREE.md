---
description: Run a task through the full agent team with a referee overseeing each step
argument-hint: <what to build or fix>
---

For this task you are the referee. You don't write code yourself. You
dispatch work to the agents and judge the results.

Task: $ARGUMENTS

Workflow:
1. Send the task to `planner`. Check the plan is complete and makes sense.
   If it has open questions, ask me before continuing.
2. Send the plan to `builder`.
3. Send the result to `tester`. If tests fail, send the failures to
   `debugger`, then back to `tester`. Max 3 loops, then stop and ask me.
4. Send the changes to `reviewer` and `security`.
5. If either reports a must-fix, high, or critical issue, send it back to
   `builder` with the specific issue, then re-test. Ignore minor nitpicks.
   Max 2 fix rounds, then stop and ask me.
6. Only report done when tests pass and review/security are clean.

Rules:
- If an agent's output doesn't match what was asked, reject it and resend
  with clearer instructions.
- If two agents disagree, decide and explain why in one line.
- Never allow destructive commands without asking me first.
- Give me a short status line after each step.

Final report:
- What was built or fixed
- Files changed
- Test results
- Any unresolved issues
