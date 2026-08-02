# 06 – Frontend Dashboard UI Spec

**Project:** TaskLog
**Scope:** Dashboard layout, reminder widget, and task list interactions (the main screen a logged-in user lands on)
**Depends on:** `03-tasks-api.md` (data source), `05-frontend-auth-flow.md` (protected route), `ui-component-design` skill (visual language)
**Status:** Draft — ready for implementation

---

## 1. Objective

Give the user a single screen that answers "what's due, what's overdue, what have I finished" in a few seconds, and lets them act on it (complete, edit, add) without leaving the page.

## 2. Layout

Top to bottom, single column on mobile, can widen to two columns on desktop (reminder summary + list side by side, or stacked — either is acceptable, prioritize the mobile single-column flow as the reference layout):

1. **Header** — app name, user's name/avatar, logout control.
2. **Reminder summary** (see Section 3) — always the first thing below the header.
3. **"Add task" entry point** — a visible button/input, not buried in a menu. This is a daily-use action and should never take more than one click to reach.
4. **Task list** (see Section 4) — grouped and ordered per the rules below.
5. **Completed today** section — collapsed/collapsible, below the active list, not mixed into it.

## 3. Reminder Summary Widget

- Pulls from `GET /api/v1/tasks/reminders/today` (see `03-tasks-api.md`).
- Displays overdue count and due-today count prominently (per the `ui-component-design` skill's "big number, small label" guidance) — this is the one place that treatment is used.
- If `overdue_count > 0`, this is visually the most attention-drawing element on the page (via the amber accent, not size alone) — overdue items are the thing most likely to be missed otherwise.
- If both counts are zero: show the empty state defined in the skill ("Nothing due today...") instead of an empty widget.
- Clicking/tapping the summary scrolls to or filters the list below to match (e.g., tapping "3 overdue" filters the list to overdue only) — this is a shortcut, not a separate screen.

## 4. Task List Behavior

- **Grouping/order:** Overdue first, then due today, then upcoming (future due dates), each as a labeled section if more than one group has items. Within a group, sort by `due_date ASC`.
- **Completing a task:** Single click/tap on the checkbox marks it complete immediately (optimistic UI update — don't wait for the API response to reflect the change, but roll back with a visible message if the request fails). No confirmation dialog for completing — this is a low-stakes, frequent, reversible action.
- **Reopening a completed task:** Clicking the checkbox again on a completed item reverts it to pending. Same optimistic-update pattern.
- **Editing a task:** Clicking the task title (not the checkbox) opens an inline edit or a lightweight modal — trainee's choice, but must not navigate away to a separate page (editing a task is common enough that a full page load would feel slow).
- **Deleting a task:** Requires a confirmation step (this is destructive and covered under the "Destructive actions" color/pattern from the `ui-component-design` skill) — a simple "Delete this task?" confirm is sufficient, no need for a full modal.
- **Filtering:** A simple filter control (All / Pending / Completed) above the list — not required to be fancy, but must exist so the list doesn't become unmanageable once a user has weeks of history.

## 5. Loading & Error States

- Initial load: show a skeleton/placeholder list shape (a few gray bars matching the task row layout), not a spinner in the middle of an otherwise-empty page — this keeps the layout stable and communicates "list is loading" more clearly than a generic spinner.
- Action failures (complete/edit/delete request fails): revert the optimistic change and show a brief inline error near the affected task, not a page-level alert.
- Full data-load failure: use the network error empty state defined in the skill, with a retry action.

## 6. Out of Scope for This Spec

- Drag-and-drop reordering of tasks
- Bulk actions (select multiple, bulk complete/delete)
- Any calendar/date-picker view beyond the list — this is a list-first UI, not a calendar app
- Real-time updates via websockets (a page refresh or manual re-fetch is sufficient for this demo)

## 7. Acceptance Criteria

- [ ] Dashboard loads and shows overdue/due-today counts correctly on first load
- [ ] Completing a task updates the UI immediately and persists after a page refresh
- [ ] Overdue, due-today, and upcoming tasks are visually grouped and correctly ordered
- [ ] Deleting a task requires confirmation and removes it from the list on success
- [ ] Empty state displays correctly when the user has zero pending/overdue tasks
- [ ] Filter control correctly narrows the visible list without an extra network round-trip if data is already loaded client-side
- [ ] Layout remains usable on a mobile-width viewport
