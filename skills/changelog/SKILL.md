---
name: changelog
description: Turn git history into human-readable release notes grouped by Breaking, Added, Changed, Fixed, Removed, and Security, and prepend them to CHANGELOG.md. Use this skill whenever the user sends /changelog, asks for release notes, "what changed since the last release", "write the changelog", "summarize commits since v1.2", is preparing a version bump or release, or needs a plain-language update for a client about what was delivered. Reads commits and diffs; never tags, commits, or pushes.
---

# Changelog

Read the commits between two points in history, work out what actually changed for the people who use the software, and write it up as release notes. The output describes impact, not commit messages.

## Scope rules

**The only file written is `CHANGELOG.md`.** Don't create git tags, bump version numbers in manifests, commit, or push. Suggest the version number; the user decides and tags it themselves.

**Never rewrite existing entries.** New notes are prepended under a new heading. Past releases are history; leave them exactly as they are, even if they're formatted badly.

**Describe from the diff when the message is useless.** Commit messages like "fix", "wip", "stuff", or "asdf" tell you nothing. Read the diff for those commits and describe what really changed. Don't copy a bad message into the changelog.

## Workflow

### 1. Find the range

```bash
git describe --tags --abbrev=0           # last tag
git tag --sort=-creatordate | head -5    # recent tags for context
```

Default range is last tag to `HEAD`. If the user named a range ("since v0.3", "since last Friday"), use theirs. If there are no tags, tell the user and use the full history, or ask for a starting commit if the history is long (over about 200 commits).

### 2. Collect the commits

```bash
git log <from>..HEAD --no-merges --pretty=format:'%h %s' --reverse
git log <from>..HEAD --merges --pretty=format:'%h %s'          # PR titles often live here
git diff <from>..HEAD --stat                                   # where the changes landed
```

For any commit with a vague message, or any commit that touches public interfaces, run `git show <hash> --stat` and read the relevant parts of the diff.

### 3. Detect breaking changes

Commit messages miss most of these. Check the diff for:

- `!` after the type (`feat!:`) or `BREAKING CHANGE` in a commit body
- Removed or renamed public functions, CLI commands, flags, or API endpoints
- Changed config file format, keys, or defaults
- Database schema migrations that aren't backward compatible
- Raised minimum versions (Rust edition, Node version, OS support)
- Changed file paths users depend on (install location, data directory)

Anything found only in the diff, not stated in a commit, goes under Breaking marked `(detected from diff, confirm)`. Under-calling a breaking change hurts users more than an extra line.

### 4. Sort into sections

Each user-visible change goes into exactly one section:

- **Breaking**: users must change something on their side to upgrade.
- **Added**: new features, commands, options.
- **Changed**: existing behavior works differently.
- **Fixed**: bugs that are gone.
- **Removed**: features taken out (also list under Breaking if anyone relied on them).
- **Security**: vulnerability fixes. Describe the class of issue and what's affected; don't publish exploit details before users have had time to update.

Internal-only work (refactors, CI, tests, formatting, dependency bumps with no behavior change, docs) is left out by default. Count it and note it in one line at the end of the entry. Include it if the user asks for a full or developer changelog.

Combine commits that are one change. Five commits building one feature become one line.

### 5. Suggest a version

Based on semver from the last tag:

- Any Breaking item: major bump (or minor bump while still on 0.x, and say so)
- Any Added item, no breaking: minor bump
- Only Fixed/Security: patch bump

State the suggestion and the reason in one line in chat. Don't write it into any manifest.

### 6. Write CHANGELOG.md

Prepend the new entry below the file's title. If `CHANGELOG.md` doesn't exist, create it with a `# Changelog` heading first. Match the existing file's style if it has one; otherwise use this:

```
## [<version>] - <YYYY-MM-DD>

### Breaking
- <what changed and what the user has to do about it>

### Added
- <feature, written as what the user can now do>

### Changed
- ...

### Fixed
- ...

### Removed
- ...

### Security
- ...

<N internal changes not listed (refactors, CI, tests).>
```

Leave out empty sections. Writing rules:

- One line per item, starting with what the user experiences, not what the code does. "Downloads resume after a dropped connection" beats "Add retry logic to fetch_chunk()".
- Breaking items say what to do: "Config key `theme` renamed to `ui.theme`. Rename it in `~/.config/app/config.toml` before upgrading."
- Short commit hashes in parentheses at the end of a line are fine for developer-facing projects. Leave them out in client mode.
- No em dashes, no marketing words ("powerful", "seamless", "exciting"), no exclamation points.

### 7. Report in chat

Show the new entry, the suggested version with its reason, and any `(detected from diff, confirm)` items the user needs to check. Then stop.

## Client mode

When the user says the notes are for a client, or sends `/changelog client`, write for someone who doesn't read code:

- No hashes, file paths, function names, or section headings like "Fixed".
- Short plain paragraphs or a simple list of what they can now do and what was repaired.
- Breaking changes become "what you need to know" with steps they can follow.
- Output in chat only, not to `CHANGELOG.md`, unless the user asks for a file. The repo changelog stays developer-facing.

## Example

From a range of 23 commits on a fictional Rust CLI:

```
## [0.5.0] - 2026-10-05

### Breaking
- `sync --force` renamed to `sync --overwrite`. Update any scripts that
  use the old flag. (3f1a9c2)
- Config moved from `~/.toolname.toml` to `~/.config/toolname/config.toml`.
  The old path is no longer read; move the file before upgrading.
  (detected from diff, confirm) (b72e014)

### Added
- `toolname status` shows pending changes without syncing. (91cd3e7)
- Progress bar for transfers over 10 MB. (c04b8f1, 7aa2d93)

### Fixed
- Sync no longer hangs when the remote drops the connection mid-file. (e5590ab)
- File timestamps are preserved on Windows targets. (2d8f6c0)

9 internal changes not listed (refactors, CI, tests).
```

Chat note alongside it: "Suggested version 0.5.0: two breaking changes, minor bump since you're pre-1.0. Please confirm the config path move in b72e014, it isn't mentioned in any commit message."

## When there's nothing user-visible

If every commit in the range is internal, say so plainly and don't write an entry. Suggest skipping the release or calling it a patch with "Internal maintenance, no user-facing changes." Don't inflate a refactor into a feature.
