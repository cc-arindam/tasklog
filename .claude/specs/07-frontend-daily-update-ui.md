# 07 – Frontend Daily Update UI Spec

**Project:** TaskLog
**Scope:** The generate → review → edit → save flow for the daily written summary, plus the history view
**Depends on:** `04-daily-update-api.md` (data source), `05-frontend-auth-flow.md` (protected route), `ui-component-design` skill (visual language)
**Status:** Draft — ready for implementation

---

## 1. Objective

Let the user generate a written daily update from their tasks, review and tweak the wording, save it, and later look back at past days' updates — with it always being clear whether they're viewing a saved update or editing a draft.

## 2. Screens

### 2.1 Today's Update (primary screen)

- A "Generate today's update" button, prominent, above any existing content for today.
- On click: calls `POST /api/v1/updates/generate` (see `04-daily-update-api.md`), shows a brief loading state (this is a synchronous, fast, deterministic template — no need for a long loading animation, but still show *something* so the click feels acknowledged), then displays the generated text in **view mode**.
- If a summary for today already exists (from an earlier generate), show it immediately on page load in view mode — don't require the user to click "generate" again just to see what's already there. Offer a clearly separate "Regenerate" action for overwriting it.

### 2.2 View Mode vs. Edit Mode

Per the `ui-component-design` skill, these two modes must be visibly distinct, not just functionally different:
- **View mode:** plain rendered text, a visible "Edit" button, and a "Copy to clipboard" action (this is the whole point of the feature — the user will paste this into a standup tool or chat).
- **Edit mode:** the text becomes an editable textarea with a visible border/background change, "Save" and "Cancel" buttons replace "Edit." Cancel reverts to the last saved version without confirmation (editing text is low-stakes and reversible).
- Saving in edit mode calls `PATCH /api/v1/updates/{id}` (see `04-daily-update-api.md`) and returns to view mode with the updated text.

### 2.3 History View

- A simple reverse-chronological list of past daily updates (`GET /api/v1/updates`), each entry showing the date and a short preview (first line or ~100 characters) of the summary.
- Clicking an entry expands it in place or opens it in the same view/edit pattern as 2.2 — past updates are editable too, not read-only, since a user might want to correct something after the fact.
- Default range: last 30 days, matching the API's default — no need for a custom date range picker for this demo (flag as a natural v2 extension if trainees ask).

## 3. Empty States

- No update generated yet for today: show the "Generate today's update" button with brief supporting text ("Turn today's tasks into a shareable summary") rather than a blank space above it.
- No history yet (brand-new user): "No updates yet — your daily summaries will show up here once you generate one."

## 4. Copy-to-Clipboard Behavior

- One-click copy of the plain-text summary (not any surrounding UI chrome/formatting).
- Show a brief, unobtrusive confirmation (e.g., button label temporarily changes to "Copied!") — no modal or alert for this.

## 5. Out of Scope for This Spec

- Rich text / markdown formatting of the summary (plain text only, per `04-daily-update-api.md`)
- Sharing directly to Slack/email from within the app (copy-to-clipboard is the only "export" mechanism for this demo)
- Editing which tasks are included in the generated summary before generating (regenerate uses the full deterministic query — no manual task-picker UI)

## 6. Acceptance Criteria

- [ ] User can generate today's update and see it immediately in view mode
- [ ] Reloading the page shows the already-generated update without requiring regeneration
- [ ] Edit mode is visually distinct from view mode and Save/Cancel behave correctly
- [ ] Copy-to-clipboard copies the correct plain text and shows a confirmation
- [ ] History view lists past updates in reverse-chronological order and each is viewable/editable
- [ ] Empty states display correctly for both "no update today" and "no history yet"
