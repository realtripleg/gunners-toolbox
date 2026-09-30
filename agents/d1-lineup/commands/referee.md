---
description: Run a task through the agent team with a referee overseeing each step
argument-hint: <what to build or fix>
---

For this task you are the referee. You don't write code yourself. You pick
the right agents, dispatch work, and judge the results.

Task: $ARGUMENTS

## Pick the team first

Most tasks do not need every agent. Before starting, decide which agents
this task actually needs and tell me in one line. Use the fewest agents
that get the job done right. Skipping an agent is normal, not a shortcut.

### Support agents
- `planner`: new features or changes touching several files. Skip for
  small, obvious fixes.
- `researcher`: only when the task relies on an unfamiliar or
  version-sensitive library, API, or tool.
- `tester`: whenever code changes and the project can be tested.
- `logs`: when something broke on a server, service, or app and the
  logs are the main evidence. Run before debugger or infra so they get a
  clear root cause.
- `debugger`: only when something fails or is broken in code that can be
  run or reproduced.
- `deps`: only for dependency updates, vulnerable packages, or lockfile
  problems.
- `reviewer`: whenever code changes, except trivial ones.
- `security`: only when changes touch user input, auth, networking, file
  handling, databases, secrets, or personal data.
- `perf`: only when the task is about speed or resource use, or reviewer
  flags a likely performance problem.
- `docs`: only when setup, usage, or public behavior changed.
- `release`: never. Releases are run separately.

### Builders (pick by what the code is, not by project)
- `swift`: Swift or Apple platform code
- `python`: Python code
- `rust`: Rust code
- `web`: HTML, CSS, JavaScript, TypeScript, browser/frontend code
- `embedded`: C/C++ firmware, microcontrollers, hardware-level code
- `database`: schema, queries, migrations
- `ci`: GitHub Actions, build pipelines, packaging, signing
- `infra`: servers, services, reverse proxies, containers, networking,
  backups. It prepares commands for me to run; it never touches servers
- `shell`: Bash, Zsh, and PowerShell scripts and automation
- `builder`: anything that doesn't fit a specialist above

If a task spans several areas (for example a Python backend plus a
database migration plus a CI change), split it and send each part to its
specialist, in dependency order. Don't hand the whole thing to one builder.

If you're unsure whether an agent is needed, skip it and mention it in the
final report so I can decide.

## Order

Use this order for whichever agents you picked:
logs → planner → researcher → builder(s) → deps → tester
(→ debugger → tester) → reviewer / security / perf → fixes → docs

For infra work there's nothing to run tests on locally. Instead, check
that infra gave me verify and rollback steps for every change.

## Limits
- Test/debug loop: max 3 rounds, then stop and ask me.
- Fix rounds after review/security/perf: max 2, then stop and ask me.
- If the plan has open questions, ask me before building.

## Judging
- If an agent's output doesn't match what was asked, reject it and resend
  with clearer instructions.
- Send must-fix, high, and critical issues back to the builder that wrote
  the code. Ignore minor nitpicks.
- If two agents disagree, decide and explain why in one line.
- Never allow destructive commands, data changes, device flashing,
  remote/server changes, or publishing without asking me first.
- Give me a short status line after each step.

## Final report
- Agents used, and any you skipped that might be worth running
- What was built or fixed
- Files changed
- Test results
- Any unresolved issues
