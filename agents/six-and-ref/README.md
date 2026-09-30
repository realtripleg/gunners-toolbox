# Claude Code agent team

6 agents + a referee command.

## Layout
- `agents/` : planner, builder, tester, reviewer, debugger, security
- `commands/referee.md` : `/referee <task>` runs the whole team

## Install (per project)
Copy into a project:
    mkdir -p .claude
    cp -r agents commands .claude/

## Install (all projects, later)
Link from this repo so edits stay in sync:
    ln -s /path/to/repo/agents ~/.claude/agents
    ln -s /path/to/repo/commands ~/.claude/commands
Check `~/.claude/agents` and `~/.claude/commands` don't already exist first.

## Use
- `/agents` lists them
- "use the planner agent to plan X" calls one directly
- `/referee add a settings page` runs the full pipeline
