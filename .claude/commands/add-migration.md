---
description: Create and apply an Alembic migration safely, with a real downgrade
argument-hint: [short description e.g. "add priority column to tasks"]
---

Create an Alembic migration for: $ARGUMENTS

Steps:
1. Run `alembic revision --autogenerate -m "$ARGUMENTS"` inside the backend container.
2. Open the generated migration file and review it before applying:
   - Confirm `upgrade()` matches the intended schema change and nothing unintended got swept in (autogenerate sometimes picks up unrelated drift — flag anything that looks unrelated to this change).
   - Confirm `downgrade()` is fully implemented — never leave it as `pass`. It should cleanly reverse everything `upgrade()` does.
   - If this migration drops a column or table, or changes a column type in a way that could lose data, stop and tell me explicitly before applying — don't run it silently.
3. Once confirmed safe, run `alembic upgrade head` inside the backend container.
4. Report what changed, and confirm `alembic downgrade -1` followed by `alembic upgrade head` again works cleanly (round-trip check).

If this migration corresponds to a change described in a spec under `.claude/specs/`, reference that spec's schema section rather than inventing column names/types yourself.
