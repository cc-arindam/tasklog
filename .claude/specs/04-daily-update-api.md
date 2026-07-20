# 04 – Daily Update API Spec

**Project:** TaskLog
**Scope:** Generating and managing the user's daily written task-update summary
**Depends on:** `01-db-setup.md` (daily_updates table), `02-auth-api.md`, `03-tasks-api.md`
**Status:** Draft — ready for implementation

---

## 1. Objective

Let a user generate a short written summary of a given day's activity — completed tasks, still-pending tasks, and anything overdue — suitable for pasting into a standup or status report. The user can edit and re-save the generated text.

## 2. Endpoints

### 2.1 `POST /api/v1/updates/generate`

**Request body:**
```json
{ "date": "2026-07-19" }
```

**Behavior:**
- Query the user's tasks for the given date: tasks with `due_date == date`, plus tasks completed on that date even if their `due_date` differs (a task due yesterday but finished today should show up as "completed today").
- Build a plain-text summary using a simple, fixed template (see 2.3) — no LLM call in this endpoint; keep it deterministic and testable for the training demo.
- Upsert into `daily_updates`: if a row for `(user_id, date)` already exists, overwrite `summary_text`; otherwise create it.

**Response (200):**
```json
{
  "id": 7,
  "update_date": "2026-07-19",
  "summary_text": "...",
  "created_at": "...",
  "updated_at": "..."
}
```

### 2.2 `PATCH /api/v1/updates/{update_id}`

**Request body:**
```json
{ "summary_text": "user-edited version of the summary" }
```

**Behavior:** Lets the user manually edit the generated text and save it. `404` if not found / not owned by current user.

### 2.3 Summary Template (deterministic, no AI call)

Plain text, generated roughly like:

```
Daily Update — {date}

Completed:
- {task title} ({n} more if list is long, otherwise list all}

Still Pending / Overdue:
- {task title} (due {due_date})

Summary: Completed {completed_count} of {total_count} tasks for {date}.
```

Keep this logic in `services/update_generator.py` as a plain string-building function — this is intentionally simple so trainees can see clean separation between "data assembly" (querying tasks) and "presentation" (formatting text), without an external AI dependency muddying the demo. (If a later session wants to swap in an actual LLM-generated summary via the Anthropic API, this function is the seam to replace — call that out as a discussion point, not something to build now.)

### 2.4 `GET /api/v1/updates`

**Query params:** `from_date`, `to_date` (optional; default to last 30 days if omitted)

**Behavior:** Returns the user's daily updates in that range, ordered by `update_date DESC` — powers the "activity history" view.

**Response (200):** array of update objects (same shape as 2.1's response).

### 2.5 `GET /api/v1/updates/{update_id}`

Single update lookup. `404` if not found / not owned by current user.

## 3. Validation Rules

- `date` in the generate request must be a valid ISO date; no restriction on future dates being disallowed, but generating for a future date will simply produce an empty/near-empty summary (no need for special-case validation — let the natural query result handle it).
- `summary_text` on `PATCH`: required, non-empty, reasonable max length (e.g., 5000 characters) to prevent abuse.

## 4. Out of Scope for This Spec

- AI/LLM-generated summaries (explicitly deferred — see note in 2.3)
- Exporting updates to PDF/email/Slack
- Multi-day "weekly summary" rollups

## 5. Acceptance Criteria

- [ ] Generating an update for a date with completed + pending tasks produces correctly categorized text
- [ ] Re-generating for the same date overwrites rather than duplicates the row (unique constraint from `01-db-setup.md` is respected)
- [ ] User can edit and save a generated summary via `PATCH`
- [ ] History endpoint returns updates within the requested date range, most recent first
- [ ] A user cannot access another user's updates
