# courtroom

Put an idea on trial. Defense and prosecution argue, 12 jurors rule GOOD or BAD, the judge counts.

## Layout
```
agents/    defendant, prosecutor, judge, juror-01 to juror-12
commands/  trial.md (main Claude, the clerk)
```

## Install
See the [main README](../README.md#install).

## Use
```
/trial <your idea>
```

## Flow
1. Main Claude asks intake questions. Answers go into a verbatim RECORD.
2. Defendant opens and may ask you questions (relayed by main Claude).
3. Prosecutor opens and may ask you questions.
4. They argue, max 20 rounds, ending early if both are done.
5. Each gives a final statement.
6. Main Claude builds one case file (idea, record, both final statements). All 12 jurors get the identical file, in parallel.
7. Each juror returns `VERDICT: GOOD|BAD` plus one line.
8. Judge counts and gives the opinion.
9. On a 6-6: jurors may ask questions, you answer, only the jury re-votes. A second tie is a final deadlock and goes back to you.

## Jurors
accountant, software engineer, attorney, product designer, security and privacy analyst, college student, marketing strategist, project manager, small business owner, industry analyst, ethics and public impact reviewer, operations and support manager.

## Notes
- Models are set to `opus` (latest Opus alias). To pin, replace with the full model string in each file's frontmatter.
- Defendant and prosecutor have web search and fetch. Jurors and judge have no tools.
- Output is saved to `trial-output/` in the directory you run it from.
- Usage is heavy: up to 40+ Opus calls in the debate, 12 per vote.
