---
description: Scaffold a new FastAPI endpoint following the project's schema/service/route pattern
argument-hint: [resource-name and verb(s), e.g. "tasks POST" or "updates GET,PATCH"]
---

Scaffold a new endpoint for: $ARGUMENTS

Follow the exact pattern already established in `backend/app/` (check an existing resource like `tasks` or `auth` for the reference pattern before writing new code):

1. **Schema** (`app/schemas/`) — Pydantic request/response models. Reuse an existing schema if one already fits instead of duplicating.
2. **Service function** (`app/services/`) — business logic lives here, not in the route handler.
3. **Route handler** (`app/api/v1/`) — thin handler that calls the service function, validates via the schema, and returns the response. Use `Depends(get_current_user)` if this endpoint requires auth (assume it does unless told otherwise).
4. **Error handling** — use the project's standard error shape (`{"detail": "..."}`) and correct HTTP status codes.
5. **Test** — add a basic pytest test in `app/tests/` covering the happy path and at least one failure case (auth failure, validation failure, or not-found, whichever applies).

Match existing naming conventions exactly (snake_case for DB/Python, plural resource names in URLs). Don't restructure existing files to accommodate this — add to what's there.

If this endpoint is covered by an existing spec in `.claude/specs/`, follow that spec instead of inventing the shape yourself, and say which spec you used.
