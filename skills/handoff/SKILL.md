---
name: handoff
description: Write HANDOFF.md capturing the exact state of a work session so the next session, the next agent, or the user next week can resume without re-discovering anything. Use this skill whenever the user sends /handoff, says "wrap up", "save where we are", "write a handoff", "I'm done for today", "pass this to the next agent", or a long coding session is ending with work half-finished. Also use /handoff resume, or any time a session starts in a repo that already has a HANDOFF.md, to pick the work back up and verify the recorded state still matches reality.
---

# Handoff

Capture the real state of the work in `HANDOFF.md` so whoever picks it up next (a new Claude session, another agent, or the user) can start working in under a minute. The file is a snapshot of state, not a diary.

## Scope rules

**Only write what actually happened.** Every claim in the file comes from this session's work or from commands run while writing it. Do not describe planned work as done. Do not say tests pass unless they were run and passed in this session. If something wasn't verified, mark it `(unverified)`.

**The only file written is `HANDOFF.md`.** Don't commit, push, stash, or clean up anything as part of the handoff. If there's uncommitted work, record it; don't tidy it.

**One file, current state.** `HANDOFF.md` is overwritten each time, not appended to. Old state that no longer matters gets dropped. Anything unresolved from the previous handoff carries forward (see step 1).

## Workflow

### 1. Read the existing handoff

If `HANDOFF.md` already exists, read it first. Every item in its "In progress", "Broken", and "Next step" sections must end up either resolved in this session or carried into the new file. Never silently drop an open item. If it was abandoned on purpose, say so in Decisions.

### 2. Collect hard state

Run these and use the output, not memory of the session:

```bash
git rev-parse --abbrev-ref HEAD          # branch
git status --short                       # uncommitted and untracked files
git diff --stat                          # size of unstaged changes
git diff --cached --stat                 # staged changes
git log --oneline -15                    # recent commits
git stash list                           # stashes someone could forget
```

If the project has a fast test or build command (under about a minute), run it and record the actual result. If it's slow, don't run it; write `Tests: not run this session` instead of guessing.

Note anything outside git that matters: a container or VM left running, a dev server on a port, a migration applied to a local database, an env var set in the shell, a file edited outside the repo (like `/etc/` config or a Proxmox host). These are the things that get lost.

### 3. Write HANDOFF.md

Write to the repo root. Keep it under about 80 lines. Plain markdown, no em dashes, no filler.

Use this structure:

```
# Handoff: <project name>

Updated: <date and time>
Branch: <branch>   Last commit: <short hash> <subject>

## Goal
<one or two sentences: what this stretch of work is trying to achieve>

## Done
- <finished thing, with file paths where useful>

## In progress
- <half-finished thing: what's written, what's missing, where it lives>

## Broken / known issues
- <what's broken, the exact error if there is one, and whether it was
  broken before this session or caused by it>

## Decisions
- <choice made> because <reason>. Rejected: <alternative> because <reason>.

## Dead ends
- <approach tried that didn't work, and why, so nobody tries it again>

## Uncommitted state
- <files changed but not committed, stashes, running services,
  out-of-repo changes>

## Next step
<one concrete action: a command to run, or a file and line to open and
what to do there. Not "continue working on X".>
```

Rules for the content:

- **Next step must be concrete.** "Finish the parser" is useless. "Open src/parser.rs:212, the `match` on `Token::Ident` has no arm for keywords yet; add it, then run `cargo test parser`" is a handoff.
- **Errors are quoted exactly.** Paste the key line of the actual error message, trimmed. Paraphrased errors can't be searched.
- **Dead ends earn their place.** This is the section that saves the most time and gets skipped the most. If an approach failed, write it down.
- **Decisions keep the reason.** A decision without a reason gets re-litigated next session.
- **Empty sections stay with "None".** Don't delete them; "None" is information.

### 4. Warn and report

Warn the user in chat if `HANDOFF.md` contains anything sensitive (hostnames, internal IPs, tokens, paths that reveal infrastructure). Suggest adding it to `.gitignore` if the repo is public or shared. Don't add it yourself.

Then give a three-line summary in chat: what's done, what's open, and the next step. Don't reproduce the file.

## Resuming (/handoff resume)

When the user sends `/handoff resume`, or a session starts in a repo with a `HANDOFF.md` and the user asks to keep going:

1. Read `HANDOFF.md`.
2. Run the same git commands from step 2 and compare. Flag every mismatch: different branch, commits that weren't there, uncommitted files that changed or disappeared, a stash that's gone. Someone (the user or another agent) may have worked in between.
3. Check anything listed under "Uncommitted state" that can be checked cheaply (is the container still running, is the port still in use).
4. Report in a few lines: the goal, whether the recorded state still matches, and the next step. Ask before starting on it if anything didn't match.

Do not trust the file over the repo. If they disagree, the repo is right and the handoff is stale.

## Example

```
# Handoff: Gun (gunlang)

Updated: 2026-10-05 22:40
Branch: feat/closures   Last commit: a91c3f2 Parse closure syntax into AST

## Goal
Add closures to Gun: parse `|x| x + 1` syntax, capture variables by
value, and call them like normal functions.

## Done
- Lexer recognizes `|` as Token::Pipe (src/lexer.rs)
- Parser builds Expr::Closure { params, body } (src/parser.rs:180-240)
- 6 new parser tests, all passing

## In progress
- Interpreter: Value::Closure variant added in src/value.rs, but
  eval for Expr::Closure is a todo!() in src/interp.rs:315

## Broken / known issues
- `cargo test` fails 1 test, caused by this session:
  interp::tests::closure_call ... panicked at 'not yet implemented'
  Expected until the eval arm is written.

## Decisions
- Capture by value (clone the Env snapshot) because Gun has no borrow
  checker and shared mutable captures would need Rc<RefCell>. Rejected:
  capture by reference, revisit if closures need to mutate outer vars.

## Dead ends
- Tried reusing Expr::Function for closures. The named-function path
  assumes a global symbol and broke recursion lookup. Separate variant
  is cleaner.

## Uncommitted state
- src/interp.rs and src/value.rs modified, not committed
- No stashes

## Next step
Open src/interp.rs:315 and replace the todo!() with: snapshot the
current Env, return Value::Closure { params, body, env }. Then add a
call path in eval_call for Value::Closure and run `cargo test interp`.
```

## Tone

Terse and factual, like notes left for a coworker you respect. No summaries of summaries, no encouragement, no "great progress today."
