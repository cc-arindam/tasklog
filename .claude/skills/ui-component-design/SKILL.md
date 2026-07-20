---
name: ui-component-design
description: Use whenever creating or restyling a React component/page in the TaskLog frontend (task cards, dashboard, reminders, daily update view, auth forms, empty/error states). Ensures new UI matches the project's established visual language instead of each component inventing its own spacing, color, and tone. Trigger on requests like "build the task card", "style the dashboard", "make the login page", "add an empty state", or any request to create/update a .jsx component.
---

# UI Component Design — TaskLog

TaskLog is a daily-use tool, not a marketing page — someone opens it many times a day to glance at what's due and check things off. The design goal is **calm speed**: a person should be able to scan pending tasks and act in seconds, without the UI competing for attention. Every choice below serves that goal.

This skill assumes Tailwind CSS utility classes (see `frontend/tailwind.config.js` if present). If the project isn't using Tailwind yet, flag that before styling rather than silently introducing a new dependency.

## Design tokens (use these, don't reinvent per component)

**Color**
- Background: `#FAFAF9` (warm off-white, not stark white — easier on the eyes for a tool used repeatedly through the day)
- Surface/card: `#FFFFFF` with a `1px solid #E7E5E4` border — avoid heavy shadows; this is a utility app, not a showcase
- Text primary: `#1C1917` / Text secondary: `#78716C`
- Status accent — pick ONE purpose per color and never reuse it for anything else:
  - Pending: `#78716C` (neutral, not alarming — most tasks are pending most of the time, it shouldn't read as a warning)
  - Overdue: `#B45309` (amber, not red — red is reserved for destructive actions like delete, never for "you're behind")
  - Completed: `#15803D`
  - Destructive actions only (delete task, sign out of all sessions): `#DC2626`

**Type**
- One typeface family for the whole app (system UI stack or a single Google Font — e.g., Inter). Do not pair a separate "display" font; this is a utility app, not editorial content. Distinctiveness here comes from spacing and hierarchy, not typographic flourish.
- Scale: 14px body, 13px secondary/meta text, 20–24px page titles. Task titles stay at body size — a to-do app where every task title is huge doesn't scan faster, it scans slower.

**Spacing & shape**
- 8px base unit. Card padding 16px, gap between list items 8px, section gaps 32px.
- `rounded-md` (6–8px) on cards and buttons — soft enough to feel approachable, not so round it feels playful/childish for what's ultimately a productivity tool.

## Component-specific guidance

**Task card / list row**
- Checkbox/status control is the leftmost, largest touch target — completing a task is the single most common action in this app and should never require precision.
- Status is shown by a small left-border accent color (2–3px) using the status colors above, not a full-card background tint — a list of 15 tasks with 15 different background colors is visually loud; a thin border accent reads at a glance without fighting for attention.
- Overdue tasks get the amber accent AND a small "3 days overdue" style label — don't rely on color alone (accessibility: color-blind users need the text).
- Completed tasks: strike-through title, reduced opacity (60%), moved to the bottom or a collapsed "Completed today" section — don't just delete them from view, since seeing what got done is part of the point of a daily-log app.

**Dashboard reminder widget**
- Lead with the number, not a paragraph: "3 overdue · 5 due today" as the first thing seen, task list below it. This is the one place in the app where a "big number, small label" treatment is actually earned — the dashboard's whole job is to answer "how am I doing today" in under a second.
- If zero pending/overdue: show a genuine empty state (see below), never just a blank space that reads as a loading glitch.

**Daily update / summary view**
- This is the one screen with a slightly more "document" feel — treat it visually distinct from the task list (e.g., serif or slightly larger line-height body text) since the user may copy this text elsewhere. Still restrained — no decoration competing with the actual words being copied.
- Edit mode and view mode should look deliberately different (e.g., a visible textarea border in edit mode) so it's never ambiguous which mode the user is in.

**Auth forms (login/signup)**
- The one place in the app that can feel slightly warmer/less utilitarian, since it's the user's first impression — but stay restrained: no illustration-heavy hero, no marketing copy. One clear headline ("Log in to TaskLog"), the form, nothing else competing.
- Inline validation errors appear directly under the relevant field, in the destructive red, with specific text ("Enter a valid email" not "Invalid input").

## Empty & error states (write these deliberately, don't default to "No data")

- No tasks today: "Nothing due today. Add a task to get started." with the add-task action right there — an empty state is an invitation to act, not just an absence notice.
- No completed tasks yet: "Nothing completed yet today" (neutral, not "You haven't done anything!" — never shame the user for an empty state).
- API/network error: explain what happened and what to do ("Couldn't load your tasks. Check your connection and try again.") — never a raw error code or stack trace in the UI.

## Consistency checklist before finishing any component

- [ ] Uses the token values above, not new one-off hex codes or spacing values
- [ ] Status color used matches its single defined purpose (pending/overdue/completed/destructive) — never reused for something else
- [ ] Color is never the only signal (text label or icon accompanies every status color)
- [ ] Responsive down to a single-column mobile layout — this app should work from a phone during a standup
- [ ] Visible keyboard focus states on every interactive element
- [ ] Empty and error states are designed, not left as a blank div