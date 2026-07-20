---
description: Generate a PR description from the current branch's diff against main, including the relevant spec's acceptance criteria
argument-hint: [optional: related spec filename, e.g. 03-tasks-api.md]
---

Generate a pull request description for the current branch compared to `main`.

1. Run `git diff main...HEAD` (and `git log main..HEAD --oneline` for commit history) to see the actual scope of the change — base the description on this, not just recent conversation.
2. Write the PR description with these sections:
   - **Summary** — 2-4 sentences on what changed and why.
   - **Changes** — bullet list of the concrete changes (new endpoints, schema changes, UI additions, etc.).
   - **Testing** — how this was tested (reference `/run-tests` output if available, or describe manual testing performed).
   - **Checklist** — if a spec filename is given ($ARGUMENTS), pull that spec's "Acceptance Criteria" section from `.claude/specs/` and reproduce it as a checkbox list reflecting what's actually been verified so far, not just what's implemented.
3. Flag anything in the diff that looks unrelated to the stated purpose of the branch, so it can be split out or explained.

Keep the tone plain and factual — no marketing language, this is for a fellow developer reviewing the code.
