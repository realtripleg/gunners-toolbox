# gunners-toolbox

Agents, commands, and skills for Claude.

```bash
git clone https://github.com/realtripleg/gunners-toolbox.git
cd gunners-toolbox
```

## Agents and commands (Claude Code)

```bash
# courtroom: put an idea on trial with /trial <idea>
cp -r courtroom/agents courtroom/commands ~/.claude/

# d1-lineup: 21 specialist agents, run with /referee <task>
cp -r d1-lineup/agents d1-lineup/commands ~/.claude/
```

## Skills

[break](skills/break), [changelog](skills/changelog), [deslop](skills/deslop), [full-audit](skills/full-audit), [grill](skills/grill), [handoff](skills/handoff). Each folder has a README that explains what the skill does.

### Claude Code

```bash
mkdir -p ~/.claude/skills
cp -r skills/* ~/.claude/skills/        # all of them
cp -r skills/grill ~/.claude/skills/    # or just one
```

Run a skill with `/<name>`, like `/grill`, or just ask for it.

### Claude apps (web and desktop)

1. Turn on **Settings > Capabilities > Code execution and file creation**.
2. Zip the skill's folder (the folder itself, not the files inside it):
   ```bash
   cd skills && zip -r grill.zip grill
   ```
   Right-clicking the folder and compressing it works too.
3. Go to **Customize > Skills**, click **+**, then **Create skill > Upload a skill**, and pick the zip.

On Team and Enterprise plans, an admin turns skills on under **Organization settings > Plugins & skills**.

## Notes

- To install into one project only, copy into that project's `.claude/` folder instead of `~/.claude/` (`mkdir -p .claude` first).
- Restart Claude Code after copying.
