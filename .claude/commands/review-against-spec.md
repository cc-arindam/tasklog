---
description: Check the current implementation against a spec's Acceptance Criteria and report what's missing
argument-hint: [spec-filename e.g. 03-tasks-api.md]
---

Review the current implementation against `.claude/specs/$ARGUMENTS`.

1. Read the spec's full contents, especially the "Acceptance Criteria" section.
2. Go through each acceptance criterion one by one. For each, actually check the relevant code (routes, models, components — whatever applies) rather than assuming from memory of past conversation. Where practical, run the relevant tests or a quick manual check to confirm behavior rather than just reading the code and assuming it works.
3. Report results as a checklist:
   - ✅ for criteria that are met, with a one-line note on how you verified it
   - ⚠️ for criteria that are partially met, with what's missing
   - ❌ for criteria that aren't implemented at all
4. Also flag anything implemented that goes *beyond* the spec's stated scope (see the spec's "Out of Scope" section) — this is worth knowing even if it's not necessarily a problem.
5. End with a short, prioritized list of what to tackle next to fully satisfy the spec.

Don't fix anything yet unless I ask — this command is for assessment, not implementation.
