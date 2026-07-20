---
description: Run backend and frontend test suites and summarize failures
argument-hint: [optional: "backend" or "frontend" to run only one side]
---

Run the test suite(s) for: $ARGUMENTS (if empty, run both backend and frontend)

Backend: `docker compose exec backend pytest -v`
Frontend: `docker compose exec frontend npm run test`

After running:
1. Summarize pass/fail counts for each suite that ran.
2. For any failures, show the specific assertion/error, not just the test name — and identify which file/function is likely responsible.
3. Group related failures together if multiple tests are failing for the same underlying reason, rather than listing them as unrelated issues.
4. Don't attempt to fix the failures yet unless I ask — just report clearly so I can decide what to tackle first.

If the containers aren't running, say so and suggest `docker compose up -d` rather than failing silently.
