# gunners-toolbox

Agents, commands, and skills for Claude Code.

## Install

Clone the repo, then copy whatever you want into `~/.claude/`.

```bash
git clone https://github.com/realtripleg/gunners-toolbox.git
cd gunners-toolbox

# courtroom: put an idea on trial with /trial <idea>
cp -r courtroom/agents courtroom/commands ~/.claude/

# d1-lineup: 21 specialist agents, run with /referee <task>
cp -r d1-lineup/agents d1-lineup/commands ~/.claude/

# skills: deslop, full-audit, grill
cp -r skills ~/.claude/
```

To install for a single project, copy into that project's `.claude/` folder instead (`mkdir -p .claude` first). Restart Claude Code after copying.
