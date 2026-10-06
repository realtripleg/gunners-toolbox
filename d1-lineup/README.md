# d1-lineup

21 specialist agents and a referee command that runs them.

## Layout
- `agents/`
  - support: planner, researcher, tester, debugger, logs, reviewer,
    security, perf, deps, docs, release
  - builders: builder (general), web, swift, python, rust, embedded,
    database, ci, infra, shell
- `commands/referee.md` : `/referee <task>` runs the team

## Install
See the [main README](../README.md#install).

## Use
- `/agents` lists them
- "use the <name> agent to <task>" calls one directly
- `/referee <task>` runs the team, picking only the agents needed
- Releases are separate: "use the release agent to prepare v1.2.0"

## Adding a specialist
1. Add `agents/<name>.md`, scoped to a language or platform, not a project
2. Add a line for it under "Builders" in `commands/referee.md`
