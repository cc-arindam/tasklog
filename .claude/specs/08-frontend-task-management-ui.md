# 08 – Frontend Task Management UI Spec

**Project:** TaskLog
**Scope:** Creating and editing tasks — the form/modal behavior referenced but not detailed in `06-frontend-dashboard-ui.md`
**Depends on:** `03-tasks-api.md` (data source), `06-frontend-dashboard-ui.md` (entry points), `ui-component-design` skill (visual language)
**Status:** Draft — ready for implementation

---

## 1. Objective

Define exactly how a user creates a new task and edits an existing one, so this doesn't get built ad hoc differently in each place it's triggered from (dashboard "Add task" button, task title click for editing).

## 2. Add Task

- Triggered from the dashboard's "Add task" entry point (`06-frontend-dashboard-ui.md`, Section 2).
- Presented as a lightweight inline form or modal — trainee's choice, but it must not be a full separate page/route. Creating a task is frequent and should feel instant.
- Fields, in this order: **Title** (required, autofocused on open), **Due date** (defaults to today), **Description** (optional, collapsed/secondary — don't make an optional field visually compete with the two required-feeling fields).
- Submitting: calls `POST /api/v1/tasks` (see `03-tasks-api.md`). On success, close the form and the new task appears in the correct group on the list (per `06`'s ordering rules) without a full page reload.
- Validation: title required — show inline error, don't allow submit with empty title. No client-side restriction on due date (past dates allowed, per the API spec).
- Cancel/close: if the user has typed a title and closes without submitting, no confirmation needed — creating a task is not destructive to lose (unlike deleting one).

## 3. Edit Task

- Triggered by clicking a task's title in the dashboard list (per `06-frontend-dashboard-ui.md`, Section 4).
- Same field set and layout as Add Task, pre-filled with the task's current values, but the submit action is "Save" rather than "Add" and calls `PATCH /api/v1/tasks/{id}` (see `03-tasks-api.md`) sending only the fields that changed (or the full object — either is fine, but don't send `status` from this form; status changes only happen via the checkbox interaction defined in `06`).
- Include a "Delete" action within this same form/modal, using the confirm-before-delete pattern already defined in `06-frontend-dashboard-ui.md` — don't duplicate a separate delete UI elsewhere.

## 3. Form Component Reuse

- Add Task and Edit Task should share a single form component (e.g., `TaskForm.jsx`) parameterized by mode (`create` | `edit`), not two separately built forms — this is a good teaching moment on component reuse vs. duplicating similar-but-slightly-different UI.

## 4. Validation & Error Handling

- Title: required, 1–255 characters (matches `03-tasks-api.md`'s validation rule) — mirror this client-side for immediate feedback, but the backend remains the source of truth.
- If the API call fails (network error, validation error from the backend despite client-side checks passing), show the error inline within the form — don't close the form and lose what the user typed.

## 5. Out of Scope for This Spec

- Recurring task creation (matches `03-tasks-api.md`'s scope boundary)
- Attachments/file uploads on tasks
- Assigning tasks to other users (this is a single-user-per-account app — no sharing/collaboration features)

## 6. Acceptance Criteria

- [ ] Add Task form creates a task and it appears correctly grouped/ordered in the dashboard list without a full reload
- [ ] Edit Task form pre-fills existing values and saves changes correctly
- [ ] Title validation prevents empty submission with a visible inline error
- [ ] Delete action from within the edit form requires confirmation and removes the task on success
- [ ] A failed save keeps the form open with the user's input intact and shows a clear error
- [ ] Add and Edit share the same underlying form component (no duplicated form code)
