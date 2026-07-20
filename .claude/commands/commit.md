---
description: Stage changes and write a commit message following the project's Git conventions
argument-hint: [optional: extra context about the change]
---

Review the current git diff (staged and unstaged) before writing anything: $ARGUMENTS

1. Run `git status` and `git diff` to see what's actually changed — don't guess based on the conversation alone, since not everything discussed necessarily ended up in the diff.
2. Stage the relevant files (`git add`), excluding anything that shouldn't be committed (`.env`, build artifacts, anything already in `.gitignore` that may have been force-added).
3. Write a commit message following `CLAUDE.md`'s convention: imperative mood, short summary line (e.g., "Add task reminder endpoint", not "Added" or "Adds"). Add a brief body only if the change isn't self-explanatory from the summary line alone.
4. Show me the exact commit message before running `git commit` — don't commit without me seeing it first.
5. If the diff spans multiple unrelated concerns (e.g., a backend feature plus an unrelated frontend fix), point that out and suggest splitting into separate commits rather than bundling them.
