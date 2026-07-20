---
description: Diagnose common docker compose failures in the TaskLog stack
argument-hint: [optional: symptom description, e.g. "backend won't start"]
---

Diagnose the current Docker Compose setup. Context from me, if any: $ARGUMENTS

Work through these checks in order and stop at the first one that explains the problem, rather than running everything blindly:

1. `docker compose ps` — are all expected services (`db`, `backend`, `frontend`) up and healthy? Note any that are restarting in a loop or exited.
2. If `db` isn't healthy: check `docker compose logs db` for startup errors, and confirm the healthcheck in `docker-compose.yml` matches how long MySQL actually takes to initialize on a fresh volume (first-time init is slower than subsequent starts).
3. If `backend` is failing: check `docker compose logs backend` — common culprits are the backend starting before `db` is healthy (check `depends_on` condition), a missing/incorrect `DATABASE_URL` in `.env`, or an unapplied migration.
4. If `frontend` can't reach the backend: confirm `VITE_API_BASE_URL` points to the right host/port for how the frontend is being accessed (browser vs. container context matters here — `localhost` inside a container isn't the host machine).
5. Port conflicts: check if `3306`, `8000`, or the frontend dev port are already in use on the host by another process.
6. Stale state: if a schema or dependency change isn't reflected, check whether a `--build` or a volume reset (`docker compose down -v`, only after confirming I'm fine losing local data) is needed.

Report which check identified the issue, the specific fix, and why it happened — so this doubles as a learning moment, not just a fix.
