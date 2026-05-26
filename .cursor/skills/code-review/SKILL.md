---
description: Perform a thorough AI code review. Use when the user asks to review code, find issues, audit a PR, or check for bugs.
---

# Code Review

Perform a structured review of code changes, focusing on correctness, security, maintainability, and alignment with project conventions.

## Review Scope

Determine the diff to review:
```bash
# Review uncommitted local changes
git diff

# Review a specific PR
gh pr diff <number>

# Review a branch against main
git diff main...<branch>
```

## Review Dimensions

Work through each dimension in order:

### 1. Correctness
- Does the code do what it claims to do?
- Are edge cases (null, empty, boundary values) handled?
- Is error handling explicit — no swallowed exceptions?
- Are async operations awaited correctly?

### 2. Security
- No hardcoded credentials, tokens, or secrets
- Input validation before use in queries or shell commands
- No SQL injection, XSS, or path traversal vectors
- Dependencies are pinned to specific versions

### 3. Test Coverage
- New functionality has corresponding tests
- Bug fixes include regression tests
- Tests cover meaningful behavior, not just happy paths

### 4. Code Style
- Follows conventions in `.cursor/rules/01-code-style.mdc`
- Functions are small and single-purpose
- Names are clear and intention-revealing

### 5. Architecture & Scope
- Change is within the stated task scope
- No unrelated files modified
- Introduces no new circular dependencies

### 6. Agent Layer Architecture (applies when diff touches `src/agents/`)
Check each constraint from `docs/architecture/agents.md`:

- **Skill state isolation** — skills read/write only skill-local state; no direct access to parent graph state
- **No adapter imports in skills** — `src/adapters/` must not be imported inside `src/agents/skills/`; ports and tools are injected
- **Skill factory signature** — every skill entry point is `build_<skill>_graph(*tools) -> CompiledGraph`; input/output types are Pydantic models from `src/domain/`
- **Tool factory signature** — every tool is `build_<name>_tool(port)` wrapping exactly one Port method
- **Parallel fan-out** — parallel skill dispatch uses `Send` + `Annotated[list[T], operator.add]` reducer; no threads or asyncio tasks
- **HITL placement** — `interrupt()` only in Elicitation skill; In-Agent HITL follows ADR-006
- **Fakes before merge** — every new Port used by a skill has a corresponding fake in `tests/fakes/` in this PR
- **Observability tags** — every new agent run sets `case_id`, `workflow_id`, `step_id` before the first LLM call

### 7. Spec coverage (applies when diff introduces a new Skill, Tool, Port, Adapter, or domain model)
For each new artifact in the diff, verify an approved spec exists in `docs/specs/` before
implementation is merged. Use the filename conventions from `docs/specs/templates/TEMPLATE-README.md`:

| New artifact type | Expected spec location |
|---|---|
| Skill | `docs/specs/skills/<skill-slug>-spec.md` |
| Tool | `docs/specs/tools/<tool-slug>-spec.md` |
| Port | `docs/specs/ports/<port-slug>-spec.md` |
| Adapter | `docs/specs/adapters/<adapter-slug>-spec.md` |
| Domain model | `docs/specs/domain/<model-slug>-spec.md` |
| Persistence | `docs/specs/persistence/<slug>-persistence-spec.md` |
| Observability | `docs/specs/observability/<slug>-observability-spec.md` |

Flag as a blocking issue if an artifact is implemented but its spec is missing.

### 8. Port & Adapter Architecture (applies when diff touches `src/ports/` or `src/adapters/`)

**Ports (`src/ports/`)** — check against `docs/architecture/integrations.md`:

- **Protocol, not ABC** — port is defined as `typing.Protocol`; no base class inheritance
- **Domain intent only (P3)** — no vendor-specific parameters on method signatures
- **No cross-port imports (P2)** — ports import from `src/domain/` only; never from `src/adapters/` or sibling ports
- **One capability per port** — no unrelated methods added to an existing port
- **Fake before merge (P5)** — a working in-memory fake exists in `tests/fakes/` in this PR

**Adapters (`src/adapters/`)** — check against `docs/architecture/integrations.md`:

- **Structural satisfaction (P4)** — adapter matches port method signatures without subclassing the Protocol; no adapter imports in agent or tool modules
- **Vendor boundary sealed (P2, P3)** — vendor SDK types and raw DTOs do not cross the adapter boundary; raw responses translated to domain models before returning
- **Domain errors only, no retries (P6)** — vendor/SDK exceptions caught and re-raised as typed domain errors from `src/domain/`; no retry, backoff, or fallback logic inside the adapter
- **Fake before merge (P5)** — matching fake in `tests/fakes/` in this PR; real external calls only under `@pytest.mark.external`

## Output Format

```
## Code Review Summary

### ✅ Passed
- <check>: <brief explanation>

### ⚠️ Suggestions (non-blocking)
- `<file>:<line>`: <issue> → <recommendation>

### ❌ Issues (must fix before merge)
- `<file>:<line>`: <issue> → <recommendation>

### Verdict
`approve` | `request changes` | `needs discussion`
```
