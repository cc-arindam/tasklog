---
description: Explicitly invoke a named subagent, bypassing automatic description-matching
argument-hint: [agent-name] [task description]
---

Explicitly invoke the subagent named in the first word of $ARGUMENTS, passing it the rest of $ARGUMENTS as its task.

Available subagents in this project (from `.claude/agents/`):
- `code-reviewer` — independent review of an implementation against `CLAUDE.md` and the relevant spec
- `db-schema-guardian` — reviews migrations/model changes against `01-db-setup.md`, checks for destructive changes and missing downgrades
- `api-contract-auditor` — cross-checks backend routes against frontend API calls for drift
- `onboarding-explainer` — explains a file/folder/pattern in plain language for a junior developer

Steps:
1. Parse the agent name from the start of $ARGUMENTS. If it doesn't match one of the names above, list the available agents above and ask which one I meant — don't guess and don't silently fall back to handling the task yourself in the main session.
2. Invoke that subagent by name explicitly (e.g., "Use the code-reviewer agent to...") so routing doesn't depend on description-matching — this command exists specifically to bypass that ambiguity.
3. Pass along the remaining text in $ARGUMENTS as the subagent's task. If no task text was given beyond the agent name, ask what I want it to look at rather than inventing a scope.
4. Once the subagent returns, relay its findings back to me in full — don't summarize away specifics like file names, line references, or its 🔴/🟡/🟢 ratings if it used them.

Example usage: `/agent code-reviewer check the changes in backend/app/api/v1/tasks.py`
