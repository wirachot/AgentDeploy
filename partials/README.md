# Partials

This directory holds shared shell snippets that are referenced from multiple
SKILL.md files. Each partial is a small, self-contained POSIX shell script
(`#!/bin/sh`, no bashisms). Keeping these snippets in one place ensures that
updates land everywhere consistently and that each SKILL.md stays focused on
its own instructions rather than re-implementing shared boilerplate.

## Invocation Model: Executable partial (emits text to stdout)

The partial is a script Claude Code's `!`...`` macro runs and substitutes its
stdout into the skill prompt. Example: `ethos-include.sh` reads `ETHOS.md` and
prints it. Skills invoke it like:

```
!`sh .adlc/partials/<name>.sh 2>/dev/null || sh ~/.claude/skills/partials/<name>.sh`
```

The consumer-project-first fallback works whether or not a copy of the partial is present in the consumer repository.

## Adding a new partial

- Keep partials POSIX-only (no bashisms, no GNU-specific flags).
- Add new partials sparingly — each one is a shared dependency that touches
  multiple skills.

