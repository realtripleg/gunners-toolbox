---
name: full-audit
description: Exhaustive security and correctness audit of an entire codebase, producing findings triaged into three severity tiers and written to audit.txt. Use this skill whenever the user sends /full-audit, or asks for a full audit, whole-codebase security review, "audit this repo", "find everything wrong with this code", or any request for a comprehensive sweep of bugs and vulnerabilities across a project rather than a single file. Report-only — never fixes code.
---

# Full Audit

Sweep an entire codebase for security vulnerabilities and bugs, triage every finding into three tiers by severity, and write the result to `audit.txt`.

## Scope rules

**This is report-only.** Do not edit, patch, refactor, or "quickly fix" anything, even when the fix is one line and obvious. The user reads the report and decides. The only file written is `audit.txt`. If the user asks for fixes afterward, that's a separate request.

**The sweep is exhaustive.** Every source file gets looked at. Do not sample, do not stop at the first 20 findings, do not skip directories because they look boring — config files, build scripts, CI definitions, and Dockerfiles are where a lot of real vulnerabilities live. On a large repo this takes many tool calls and a long time; that is expected and correct. If context is filling up, write partial findings to `audit.txt` incrementally rather than dropping coverage.

Excluded by default (note the exclusion in the report, don't audit): `node_modules/`, `vendor/`, `.venv/`, `target/`, `dist/`, `build/`, `.git/`, minified bundles, and lockfiles (lockfiles are read for dependency versions, not audited as code).

## Workflow

### 1. Map the codebase

Before reading anything in depth, get the shape of it:

```bash
# file inventory by type and size
find . -type f -not -path './.git/*' -not -path './node_modules/*' -not -path './vendor/*' \
  -not -path './.venv/*' -not -path './target/*' | sed 's/.*\.//' | sort | uniq -c | sort -rn
```

Identify: languages, frameworks, entry points (main, server, handlers, routes, CLI parsers), anything that touches the network, the filesystem, a database, a shell, or user input. Those get read first and read closely — they're where exploitable bugs concentrate.

Record the total file count. The report states how many files were audited so the user can tell if coverage was real.

### 2. Fast pattern pass

Grep for the cheap wins across the whole tree before manual reading. These are leads, not findings — every hit gets confirmed by reading the surrounding code, because most of them will be false positives.

```bash
# hardcoded secrets
grep -rniE '(api[_-]?key|secret|passwd|password|token|private[_-]?key|bearer)\s*[:=]\s*["\047][^"\047]{8,}' --exclude-dir={.git,node_modules,vendor,.venv,target}
# AWS keys, generic high-entropy assignments, private key blocks
grep -rnE 'AKIA[0-9A-Z]{16}|-----BEGIN [A-Z ]*PRIVATE KEY-----' --exclude-dir={.git,node_modules,vendor}
# shell/command execution
grep -rnE 'system\(|popen\(|exec\(|execve|subprocess\.|os\.system|child_process|Runtime\.getRuntime|shell=True|eval\(' --exclude-dir={.git,node_modules,vendor}
# SQL string building
grep -rniE '(select|insert|update|delete).*(\+|\$\{|%s|f["\047]|\.format\()' --exclude-dir={.git,node_modules,vendor}
# deserialization
grep -rnE 'pickle\.loads|yaml\.load\(|Marshal\.load|unserialize\(|ObjectInputStream|JSON\.parse\(.*req' --exclude-dir={.git,node_modules,vendor}
# TLS / crypto weakness
grep -rniE 'verify\s*=\s*False|InsecureSkipVerify|CERT_NONE|md5|sha1\(|DES|ECB|Math\.random' --exclude-dir={.git,node_modules,vendor}
# debug and permissive config
grep -rniE 'DEBUG\s*=\s*True|Access-Control-Allow-Origin.*\*|0\.0\.0\.0|chmod\s+777|allow_all' --exclude-dir={.git,node_modules,vendor}
```

Adapt the patterns to the languages actually present. Also check `git log -p --all -S 'password' --pickaxe-regex` style history searches if secrets appear to have been removed — a rotated-out key still in history is a live finding.

### 3. Read the code

The grep pass will not find logic bugs, broken authorization, or race conditions. Read the code, prioritizing:

**Security**
- Input validation on every trust boundary: HTTP handlers, CLI args, env vars, file parsers, IPC, message queues
- Authentication and session handling: token generation randomness, expiry, revocation, fixation, comparison timing
- Authorization: is every privileged action checked, or does one endpoint trust a client-supplied user ID
- Injection: SQL, command, LDAP, template, header, log, XPath, NoSQL
- Path traversal and arbitrary file read/write, including archive extraction (zip slip) and upload handlers
- SSRF: any place a user-supplied URL is fetched
- XSS, CSRF, clickjacking, open redirect on anything web-facing
- Crypto: hardcoded IVs, ECB mode, weak hashing for passwords, custom crypto, missing signature verification
- Secrets management: committed credentials, secrets in logs, secrets in error messages
- Dependencies: pinned versions against known-vulnerable releases; flag anything unmaintained or fetched over HTTP
- Denial of service: unbounded allocation, regex catastrophic backtracking, missing rate limits, zip bombs

**Correctness**
- Memory safety in C/C++ and `unsafe` Rust: bounds, lifetimes, use-after-free, integer overflow before allocation
- Concurrency: shared mutable state without locks, TOCTOU, deadlock ordering, non-atomic check-then-act
- Error handling: swallowed exceptions, ignored return values, errors that fail open instead of closed
- Resource leaks: unclosed files, sockets, connections; missing cleanup on the error path
- Off-by-one, null/nil dereference, unhandled edge cases in parsing
- Logic bugs: inverted conditions, wrong operator, copy-paste errors between similar branches

### 4. Triage

Every finding lands in exactly one tier, ranked by **severity and exploitability only**. Fix effort is irrelevant to the tier — a two-week fix for a remote RCE is still Tier 1.

**Tier 1 — critical.** Exploitable by an unauthenticated remote attacker, or causes data loss, or exposes live credentials. Auth bypass, RCE, SQL injection on a public endpoint, a committed production key, memory corruption reachable from network input. Assume it is being exploited.

**Tier 2 — serious.** Real vulnerability or real bug, but needs preconditions: authenticated access, local access, a specific race, an unusual input. Also: crashes, corruption, and privilege escalation between existing user roles.

**Tier 3 — hardening and latent issues.** Defense-in-depth gaps, weak-but-not-broken crypto, missing security headers, unbounded resource use with no clear attack path, dead error handling, bugs that can't currently be reached but will bite after a refactor.

When tier placement is genuinely ambiguous, place it in the higher tier and say why it might belong lower. Under-calling a real vulnerability costs more than an extra line in the report.

Do not pad the report. A finding with no described mechanism of harm is noise — cut it or make the harm explicit.

### 5. Write audit.txt

Write to `audit.txt` in the repo root. Plain text, no markdown syntax, since this is a .txt read in a terminal.

Warn the user in chat, not just in the file: **audit.txt will contain file paths, code snippets, and possibly secret material. Do not commit it.** Suggest adding it to `.gitignore`. If `audit.txt` already exists, say so and confirm before overwriting.

Use this exact structure:

```
FULL AUDIT - <repo name>
Generated: <date>
Files audited: <N>   Excluded: <list>
Languages: <list>

SUMMARY
  Tier 1 (critical): <n>
  Tier 2 (serious):  <n>
  Tier 3 (hardening):<n>

================================================================
TIER 1 - CRITICAL
================================================================

[1.1] <one-line title>
  File:     path/to/file.py:142
  Category: SQL injection
  Impact:   Unauthenticated attacker can read or modify the entire
            users table via the search parameter.
  Detail:   <what the code does, why it's wrong, how it's reached>
  Evidence: <the relevant lines, trimmed>
  Fix:      <what to change, in one or two sentences - description only>

[1.2] ...

================================================================
TIER 2 - SERIOUS
================================================================
...

================================================================
TIER 3 - HARDENING
================================================================
...

COVERAGE NOTES
  <anything not fully audited, and why. Generated files, opaque
  binaries, code requiring runtime context to evaluate.>
```
Here's an example (a fictional project):
```
FULL AUDIT - example/pastebin-lite
Generated: 2026-01-15
Files audited: 64 source/config files out of 71 tracked
Excluded: static/vendor/ (bootstrap.min.css, htmx.min.js), uv.lock (read
          for dependency versions only), .git/
Languages: Python, HTML (Jinja2), YAML (1 workflow), Dockerfile

Project shape: a small Flask paste-sharing app backed by SQLite, deployed as
a single container. Anonymous users can create and search public pastes;
logged-in users can create private pastes and upload zip archives of files.


SUMMARY
  Tier 1 (critical): 1
  Tier 2 (serious):  2
  Tier 3 (hardening):2


================================================================
TIER 1 - CRITICAL
================================================================

[1.1] SQL injection in public paste search
  File:     app/routes/search.py:41
  Category: SQL injection
  Impact:   Any unauthenticated visitor can read the entire database,
            including the users table (emails and password hashes) and the
            contents of every private paste.
  Detail:   The `q` query parameter is formatted straight into the SQL
            string. The endpoint has no login requirement and is linked from
            the homepage search box. SQLite's UNION support makes extraction
            trivial: ?q=' UNION SELECT email, password_hash, 1 FROM users--
  Evidence: sql = f"SELECT id, title, created FROM pastes WHERE public = 1
                     AND title LIKE '%{q}%'"
            rows = db.execute(sql).fetchall()
  Fix:      Use a parameterized query (`... LIKE ?` with `f"%{q}%"` passed as
            a parameter). The other 11 queries in the app already do this;
            this is the only one built with an f-string.


================================================================
TIER 2 - SERIOUS
================================================================

[2.1] Zip upload extracts outside the upload directory
  File:     app/routes/upload.py:57-63
  Category: Path traversal (zip slip)
  Impact:   A logged-in user can upload a zip with an entry named
            ../../app/templates/base.html and overwrite application files,
            which gives them stored XSS on every page and, via a Jinja
            template, code execution in the container.
  Detail:   ZipFile.extractall() is called on the user's archive with the
            destination set to uploads/<user_id>/. Entry names are never
            checked. Requires an account, which is why this is Tier 2, but
            registration is open to anyone with an email address.
  Evidence: with zipfile.ZipFile(f) as z:
                z.extractall(os.path.join(UPLOAD_DIR, str(user.id)))
  Fix:      Resolve each entry's target path and reject any that does not
            stay under the user's upload directory before writing it.

[2.2] Password reset tokens never expire and are not single-use
  File:     app/auth/reset.py:22-48
  Category: Authentication
  Impact:   Anyone who gets hold of an old reset email (a shared inbox, a
            forwarded message, a breached mail account) can take over the
            account at any time in the future, even after the owner has
            reset their password since.
  Detail:   The token is stored in the users table with no timestamp and is
            not cleared after a successful reset.
  Fix:      Store an issued-at time, reject tokens older than an hour, and
            delete the token when it is used.


================================================================
TIER 3 - HARDENING
================================================================

[3.1] No rate limit on /login
  File:     app/auth/login.py
  Detail:   Unlimited password guesses per account and per IP. Passwords are
            hashed with argon2, so offline cracking is not the concern;
            online guessing against weak passwords is.

[3.2] Session cookie missing the Secure flag
  File:     app/__init__.py:18
  Detail:   SESSION_COOKIE_SECURE is not set. The app is served behind TLS in
            production, so this only matters if a user ever loads it over
            plain http, but it costs one line to close.


COVERAGE NOTES

  Read in full: all 52 Python files, all 9 templates, the Dockerfile,
  pyproject.toml, and .github/workflows/ci.yml.

  Not evaluated: static/vendor/ (unmodified upstream releases, versions
  checked against known advisories, none affected).

  Git history was searched for removed secrets (password, api_key, AKIA,
  SECRET_KEY). The Flask SECRET_KEY was committed in an early revision and
  later moved to an env var; the current production value differs from the
  committed one, so this is not listed as a live credential.
```
Number findings within tiers (1.1, 1.2, 2.1...) so the user can reference them later.

### 6. Report in chat

After writing the file, give a short summary in chat: the tier counts, the Tier 1 findings by title with file paths, and the path to `audit.txt`. Do not reproduce the whole report in the conversation — that's what the file is for. Then stop; don't start fixing.

## When there's nothing in Tier 1

Say so plainly. An empty Tier 1 on a small, careful codebase is a legitimate result. Do not promote a Tier 2 finding to fill the space, and do not manufacture Tier 3 filler to make the report look thorough. State the coverage numbers and let the emptiness speak.
