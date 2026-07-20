---
description: Scaffold a new numbered spec file in .claude/specs/ with the project's standard section structure
argument-hint: [short-topic-name e.g. "notifications"]
---

Create a new spec file in `.claude/specs/` for the topic: $ARGUMENTS

First, look at the existing files in `.claude/specs/` to determine the next sequential number (e.g., if `01`–`05` exist, this one is `06`).

Name the file `NN-<kebab-case-topic>.md` (e.g., `06-notifications.md`).

Use this exact section structure, matching the style of the existing specs in the folder:

```
# NN – <Title> Spec

**Project:** TaskLog
**Scope:** <one-line scope statement>
**Depends on:** <list any spec files this builds on, or "None">
**Status:** Draft — ready for implementation

---

## 1. Objective

## 2. Endpoints (or Components / Schema, whichever fits the topic)

## 3. Validation Rules

## 4. Out of Scope for This Spec

## 5. Acceptance Criteria
```

Fill in a reasonable first draft for each section based on the topic and what you can infer from `CLAUDE.md` and the existing specs — but keep it clearly marked as a draft for me to review and edit, not a finished spec. Ask me any clarifying questions about scope before going too deep into detail, rather than guessing extensively.
