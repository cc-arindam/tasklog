---
description: Implement a spec file end-to-end, following CLAUDE.md conventions
argument-hint: [spec-filename e.g. 02-auth-api.md]
---

Read `.claude/specs/$ARGUMENTS` in full, along with `CLAUDE.md` at the project root.

Before writing any code:
1. List out the endpoints/components/tables this spec requires, in the order you'll build them.
2. Note any dependencies on other specs (check the spec's own "Depends on" line) — if a dependency isn't implemented yet, stop and tell me before proceeding.
3. Flag anything in the spec that seems to conflict with an existing implementation, rather than silently overriding either one.

Then implement the spec exactly as written:
- Follow the folder structure, naming conventions, and API/DB conventions from `CLAUDE.md` — don't introduce new patterns not already established in the codebase.
- Stay within the spec's stated scope — do not build anything listed under that spec's "Out of Scope" section, even if it seems like a natural addition.
- Write or update tests alongside the implementation where the spec's Acceptance Criteria implies testable behavior.

When finished, go through the spec's "Acceptance Criteria" checklist item by item and report which are satisfied, which aren't, and why.
