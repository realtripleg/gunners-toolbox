# full-audit

A full security and bug audit of an entire codebase. Run `/full-audit` and Claude maps the repo, greps for common vulnerability patterns, reads the code, and writes every finding to `audit.txt` in three tiers: critical, serious, and hardening. Each finding has the file and line, the impact, the evidence, and a suggested fix.

Report only: it never changes your code. Don't commit `audit.txt`, since it can contain file paths and secrets.

On a big repo this takes a long time and a lot of tokens.
