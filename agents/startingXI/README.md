# Claude Code agent team

11 agents + a referee command.

## Layout
- `agents/`
  - core: planner, builder, tester, reviewer, debugger, security
  - extra: researcher, web, docs, perf, release
- `commands/referee.md` : `/referee <task>` runs the whole team

## Install (per project)
    mkdir -p .claude
    cp -r agents commands .claude/
## Use
- `/agents` lists them
- "use the <name> agent to <task>" calls one directly
- `/referee <task>` runs the full pipeline
- Releases are separate: "use the release agent to prepare v1.2.0"
