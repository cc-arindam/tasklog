# 03 – Tasks API Spec

**Project:** TaskLog
**Scope:** CRUD endpoints for tasks, plus the "pending/reminder" query used by the dashboard
**Depends on:** `01-db-setup.md` (tasks table), `02-auth-api.md` (auth dependency)
**Status:** Draft — ready for implementation

---

## 1. Objective

Let an authenticated user create, view, update, complete, and delete their own tasks, and fetch a reminder-style list of tasks pending for today (and overdue).

All endpoints in this spec require authentication (`Depends(get_current_user)`) and only ever operate on the current user's own tasks — never expose or allow access to another user's tasks, even by guessing an ID.

## 2. Endpoints

### 2.1 `POST /api/v1/tasks`

**Request body:**
```json
{
  "title": "Write project proposal",
  "description": "Draft outline for the Q3 proposal",
  "due_date": "2026-07-20"
}
```

**Behavior:**
- `status` defaults to `pending` — not accepted in the create payload.
- `user_id` is taken from the authenticated user, never from the request body.

**Response (201):** the created task object (see 2.6 for shape).

### 2.2 `GET /api/v1/tasks`

**Query params (all optional):**
- `status`: filter by `pending` | `in_progress` | `completed`
- `due_date`: filter to a specific date (`YYYY-MM-DD`)
- `from_date` / `to_date`: filter to a date range

**Behavior:** Returns only the current user's tasks, ordered by `due_date ASC, created_at ASC`. No filters = return all of the user's tasks (paginate if the list is large — see Section 4).

**Response (200):** array of task objects.

### 2.3 `GET /api/v1/tasks/{task_id}`

Returns a single task. `404` if it doesn't exist or doesn't belong to the current user (don't distinguish between the two cases in the response — same `404` either way).

### 2.4 `PATCH /api/v1/tasks/{task_id}`

**Request body (all fields optional):**
```json
{
  "title": "...",
  "description": "...",
  "due_date": "2026-07-21",
  "status": "completed"
}
```

**Behavior:**
- If `status` changes to `completed`, set `completed_at = now()` in the service layer.
- If `status` changes away from `completed` to anything else, clear `completed_at` back to `null`.
- `404` if task not found / not owned by current user.

**Response (200):** the updated task object.

### 2.5 `DELETE /api/v1/tasks/{task_id}`

Hard delete. `204` on success, `404` if not found / not owned by current user.

### 2.6 Task Object Shape (used across all responses above)

```json
{
  "id": 12,
  "title": "Write project proposal",
  "description": "Draft outline for the Q3 proposal",
  "status": "pending",
  "due_date": "2026-07-20",
  "completed_at": null,
  "created_at": "2026-07-18T09:14:00Z",
  "updated_at": "2026-07-18T09:14:00Z"
}
```

### 2.7 `GET /api/v1/tasks/reminders/today`

Purpose-built endpoint for the dashboard reminder widget.

**Behavior:** Returns tasks where:
- `due_date <= today` AND `status != completed` (this naturally includes overdue tasks, not just today's)

Ordered by `due_date ASC` so the most overdue items surface first.

**Response (200):**
```json
{
  "overdue_count": 2,
  "due_today_count": 3,
  "tasks": [ /* array of task objects, overdue first */ ]
}
```

This is a read-only convenience endpoint — it does not replace `GET /tasks?status=pending`, it's specifically shaped for the reminder UI so the frontend doesn't have to compute overdue-vs-today counts itself.

## 3. Validation Rules

- `title`: required on create, 1–255 characters.
- `due_date`: required on create, must be a valid ISO date. No restriction against past dates (users may want to log a task retroactively) — don't add "due_date must be today or future" validation.
- `status`: must be one of the three enum values; reject anything else with `422`.

## 4. Pagination (keep simple for the demo)

- `GET /tasks` supports `limit` (default 50, max 200) and `offset` query params.
- Response includes a `total` count alongside the array when paginated:
  ```json
  { "total": 87, "items": [ /* tasks */ ] }
  ```
  Note this changes the plain-array shape from 2.2 — if a trainee spots this inconsistency, that's a good discussion point about designing consistent API envelopes from the start rather than adding pagination as an afterthought. For this demo, keep `GET /tasks` returning the paginated envelope shape consistently (not a bare array) to avoid the mismatch.

## 5. Out of Scope for This Spec

- Recurring tasks / repeat schedules
- Task categories, tags, or priority levels (fine to mention as a "v2" extension exercise for trainees)
- Bulk operations (bulk complete, bulk delete)
- Push/email delivery of reminders — this spec only provides the data endpoint the frontend polls or fetches on load

## 6. Acceptance Criteria

- [ ] A user can create a task and immediately fetch it back
- [ ] A user cannot view, edit, or delete another user's task (`404`, not `403`, to avoid confirming existence)
- [ ] Marking a task `completed` sets `completed_at`; reverting clears it
- [ ] `/tasks/reminders/today` correctly separates overdue vs. due-today counts
- [ ] Filtering by `status` and date range on `GET /tasks` returns correct results
- [ ] Pagination params work and `total` reflects the unfiltered/filtered count correctly
