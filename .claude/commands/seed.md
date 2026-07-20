---
description: Run (or create, if missing) the local dev database seed script
argument-hint: [optional: "reset" to wipe and reseed, or a specific dataset name]
---

Handle database seeding for local development: $ARGUMENTS

1. Check whether `backend/app/db/seed.py` exists (this is defined in `01-db-setup.md` — a demo user, a handful of sample tasks across past/today/tomorrow due dates, and one sample daily update).
   - If it doesn't exist yet, create it following that spec exactly: don't invent a different demo dataset shape.
   - If it exists, don't rewrite it from scratch — just run it, unless I've asked for a change to the data it creates.

2. Before running anything, confirm which environment this is targeting. This script must only ever run against the **local Docker `db` service** — refuse to run it (and say so clearly) if `DATABASE_URL` or the current context looks like it's pointing at anything other than local Docker.

3. If `$ARGUMENTS` includes "reset":
   - Warn me that this will wipe existing local data before proceeding.
   - Run `alembic downgrade base` then `alembic upgrade head` (or truncate the relevant tables) to get a clean schema, then run the seed script.
   - Don't run a destructive reset without this explicit confirmation step, even if I passed "reset" as an argument — restate what will be wiped and proceed.

4. Run the seed script inside the backend container:
   `docker compose exec backend python -m app.db.seed`

5. After running, report what was created (user email/credentials for local login, task count and their due dates, whether the daily update sample was created) so I can immediately log in and see data without querying the DB manually.

If `$ARGUMENTS` names a specific dataset (e.g., "overdue-heavy" or "empty-state") that isn't part of the default seed script, ask whether I want a variant/flag added to `seed.py` rather than creating a separate one-off script.