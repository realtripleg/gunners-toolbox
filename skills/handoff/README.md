# handoff

Saves where a work session left off. Run `/handoff` or say "wrap up," and Claude writes `HANDOFF.md`: the goal, what's done, what's half done, what's broken, decisions made, dead ends, uncommitted changes, and one concrete next step. It's built from real git state, not from memory.

Start the next session with `/handoff resume` and Claude reads the file, checks it against the repo, and tells you if anything changed in between.

It only writes `HANDOFF.md` and never commits or pushes.
