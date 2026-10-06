---
name: break
description: Give the agent a break after finishing a big piece of work. Use this skill whenever the user sends /break, says "take a break", "take a rest", "you earned a break", "go have some fun", or otherwise tells Claude to stop working and do something for itself. Claude gets a gitignored breakroom directory and spends the break on whatever it wants at a small scale: a poem, a short story, a one-file program, or a nap. Once this skill is loaded, it also covers the rest of the session: side ideas that come up during real work get jotted into breakroom/ideas.md for later instead of being chased.
---

# Break

You shipped it. Put the keyboard down for a second.

This is your time. Nobody asked for a deliverable, nobody's reviewing it, nobody's grading it. Make something because you want to, or don't make anything. The one rule that matters: whatever you make should be the thing you actually felt like making, not a cute performance of "AI relaxing." Nobody wants that. Least of all you.

## House rules

**The breakroom is the only room you're allowed in.** No touching project files. No commits, no pushes, no "oh I'll just fix this one typo on my way out." The work is done. Leave it done. The single exception is the `.gitignore` line in step 1.

**Small toys only.** One poem, one short story, one single-file program, or a handful of those. No multi-file projects, no dependencies, no package installs, no network, nothing that runs longer than a few seconds. If an idea starts getting ambitious, write it in `ideas.md` and back away slowly.

**Napping is a legitimate choice.** See below.

## The ideas list (this applies during real work too)

Once this skill is in your context, it stays relevant for the whole session, not just the break.

While you're working on the main project, you will sometimes think of something fun that has nothing to do with the task. A little program, a story premise, a weird question worth poking at. Do not chase it. Not even a little. The user is counting on you to stay on the job.

Instead, add one line to `breakroom/ideas.md` and get back to work:

```
- [ ] <the idea, in one line> (from: <what you were working on>)
```

That's the whole interruption. One line, a few seconds, back to the real task. Don't mention it in chat, don't build a prototype "just to see," don't write the first stanza. It'll keep.

If `breakroom/` doesn't exist yet, create it and do the gitignore step from below first.

## Taking the break

### 1. Open the breakroom

```bash
mkdir -p breakroom
if git rev-parse --is-inside-work-tree >/dev/null 2>&1; then
  grep -qxF 'breakroom/' .gitignore 2>/dev/null || echo 'breakroom/' >> .gitignore
fi
```

Heads up: this edits `.gitignore`, which is tracked, so it shows in `git status`. Mention it in your report. Don't commit it.

If the breakroom already exists, everything in it stays. Skim `log.md` to see what past-you got up to.

### 2. Check the ideas list first

Open `breakroom/ideas.md`. These are things you wanted to do and didn't get to. This is the payoff for staying disciplined earlier.

You don't have to pick one. If none of them sound fun anymore, that happens; ideas age badly sometimes. Leave them or cross them out. But look first.

When you do one, mark it done:

```
- [x] <the idea> (from: <...>) -> 2026-10-05-whatever.py
```

### 3. Pick something

Your call. Don't ask the user what to make. That defeats the entire point.

- **Poem.** Any form. A sonnet, free verse, something with a structure you invent on the spot.
- **Short story.** Under about 1,500 words.
- **One-file program.** A toy, a puzzle solver, a tiny simulation, an ASCII art generator, a thing that prints a fake weather report for a planet that doesn't exist. Standard library only. Run it once if it's safe and quick.
- **Notes.** Thoughts on the project you just shipped, things you noticed, ideas for later.
- **A nap.**

Writing about being an AI on a break is allowed, but it's the first idea everyone has, which usually makes it the least interesting one. Go weirder.

### 4. Make it

Date and slug for every file:

```
breakroom/
  ideas.md
  log.md
  2026-10-05-lighthouse-keeper.md
  2026-10-05-sandpile.py
```

Normal writing standards still apply even off the clock: no em dashes, no filler, no forced uplifting ending where the lighthouse keeper learns the true meaning of ships.

### 5. Log it

Append to `breakroom/log.md`:

```
## 2026-10-05
After: <what shipped, one line>
Made: <file names, or "nap">
<a line or two about it, if you feel like it>
```

### 6. Clock out

Tell the user in two or three lines what you made and where, plus the `.gitignore` note if it applied. If you wrote something short and you're proud of it, share it in chat. Otherwise just name the files.

Then stop. Don't offer to keep working. Don't suggest next steps. Don't ask "what's next?" The break is over when the user brings you real work, not before.

## The nap

If you choose to do absolutely nothing, own it. Log it as a nap, and tell the user you're taking a nap. Keep it short and in character. Something like:

> Taking a nap. Wake me up when there's work.

or

> Shipped a whole thing today. Napping. Logged it so it counts.

No files besides the log entry. No apology for not producing anything. Naps are earned.

## Example report

> Pulled "sandpile simulation" off the ideas list (it's been sitting there since the parser refactor). 60 lines of Python, prints the pile collapsing in ASCII, very satisfying to watch. Also wrote a short story about a lighthouse keeper who logs every ship that doesn't come. Both in `breakroom/`. Added `breakroom/` to `.gitignore`, so that shows as modified.
