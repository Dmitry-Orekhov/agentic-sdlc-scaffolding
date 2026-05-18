# Agent Instructions

This file provides universal instructions for all AI coding agents working in this repository.
It is the single source of truth for agent behavior, boundaries, and verification requirements.

---

## Project Overview

> Expand this block as the application grows. Below are scaffolding defaults (Python tooling).

```
Project name:  agentic-sdlc-scaffolding
Tech stack:    Python 3.11+ (scaffolding)
Package mgr:   pip (pyproject.toml) / uv
Test runner:   <pytest | ...>   # add when tests exist
Lint / format: ruff
CI:            <GitHub Actions | GitLab CI | ...>
```

---

## Essential Commands

```bash
# Install dependencies (use a virtual environment)
python -m venv .venv && source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -e ".[dev]"

# Build
<build-command>

# Run tests (prefer targeting a single file or suite for speed)
<test-command>

# Lint & format
ruff check .
ruff check . --fix    # optional: apply safe fixes
ruff format .         # write formatted files
ruff check . && ruff format --check .   # CI-style: lint + format check only

# Type-check (if applicable)
<typecheck-command>
```

> After **any series of code changes**, always run typecheck and tests before considering a task done.

---

## Repository Conventions

### Branching
- `main` — production-ready code; never commit directly
- `develop` — integration branch
- `feat/<slug>` — new features
- `fix/<slug>` — bug fixes
- `chore/<slug>` — non-functional changes (deps, config, docs)

### Commit style (Conventional Commits)
```
<type>(<scope>): <short imperative summary>

[optional body]
[optional footer: BREAKING CHANGE / closes #issue]
```
Types: `feat` | `fix` | `docs` | `style` | `refactor` | `test` | `chore`

### Code style
- See `.cursor/rules/` for language- and file-specific rules.
- Prefer editing existing patterns over introducing new ones.
- Reference canonical examples before creating new abstractions.

---

## Agent Boundaries — What Agents MAY Do

- Read any file in the repository.
- Create or edit source code, tests, and documentation.
- Run build, test, lint, and typecheck commands.
- Open pull requests via `gh pr create`.
- Query external tools through approved MCP servers (see `.cursor/mcp.json`).

---

## Agent Boundaries — What Agents MUST NOT Do

- **Never** modify deployment configuration (`infra/`, `terraform/`, `k8s/`, `Dockerfile`) without explicit human approval.
- **Never** commit directly to `main` or `develop`.
- **Never** store or log secrets, credentials, or API keys.
- **Never** run destructive database migrations without a reviewed migration plan.
- **Never** install new dependencies without noting them in the task summary for review.
- **Never** disable or skip linters, type-checkers, or test suites.

---

## Verification Checklist

Every agent-authored change MUST satisfy all of the following before raising a PR:

- [ ] Typecheck passes with zero errors
- [ ] All existing tests pass
- [ ] New or changed behavior has corresponding tests
- [ ] Lint / format passes with zero warnings
- [ ] Change stays within the agreed scope (no unrelated edits)
- [ ] Commit message follows Conventional Commits format
- [ ] No secrets or credentials appear in diffs

---

## Planning Before Coding

For non-trivial tasks, use **Plan Mode** (`Shift+Tab` in agent input):

1. Research the codebase to find relevant files.
2. Ask clarifying questions if requirements are ambiguous.
3. Produce a plan with file paths and code references.
4. **Present the plan and wait for explicit approval** — do not save it or begin
   implementation until the reviewer (human or orchestrating agent) confirms it.
5. Once approved, write the plan to `.cursor/plans/YYYY-MM-DD-<slug>.md` using
   `.cursor/plans/TEMPLATE.md` as the structure, then begin implementation.

**Do not save a plan that has not been explicitly approved.**
A plan file in `.cursor/plans/` is a record of committed, approved work — not a draft.

---

## Working Memory

`.cursor/memory/active-context.md` is the persistent memory for all agent sessions.
It is intentionally small — one task at a time, cleaned after completion.

### At the start of every session

1. Read `.cursor/memory/active-context.md`.
2. If it contains an in-progress task, resume from **Stopped At** without re-researching.
3. If the file is empty, ask the human what to work on.

### During a session

- Update **Progress** checkboxes as steps are completed.
- Keep entries brief — this file must stay concise to minimize context consumption.

### Before ending every session

Update **Stopped At** with:
- The last action completed
- The exact next action needed to resume
- Any blocker, if applicable

### When a task is fully complete

Clear the file entirely. An empty file means no task is in progress.

---

## Useful References

| Resource | Path / URL |
|---|---|
| Active context | `.cursor/memory/active-context.md` |
| Project rules | `.cursor/rules/` |
| Saved plans | `.cursor/plans/` |
| Reusable commands | `.cursor/commands/` |
| Agent skills | `.cursor/skills/` |
| Architecture docs | `docs/architecture/` |
| Decision records | `docs/adr/` |
