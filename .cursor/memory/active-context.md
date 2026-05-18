# Active Context

Persistent memory for the current agent session.
Read this at session start. Clear this when the task is complete.

---

## Current Task

```
Task:    <title or issue reference, e.g. "feat: add OAuth login" / #42>
Branch:  <git branch>
Plan:    <path to approved plan, e.g. .cursor/plans/2026-05-18-oauth.md>
Status:  in-progress | blocked | awaiting-review
```

## Progress

- [x] <completed step>
- [ ] <current step>
- [ ] <next step>

## Stopped At

```
Last action:  <what was just completed>
Next action:  <exactly what to do next to resume>
Blocker:      <if blocked — what is needed to unblock, or leave empty>
```

---

> Clear this file completely when the task is done and committed.
> A clean file means no task is in progress.
