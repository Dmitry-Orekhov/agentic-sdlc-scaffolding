# Quality and delivery

**Parent:** [OVERVIEW.md](OVERVIEW.md)

---

## Testing strategy

| Layer | Scope |
|---|---|
| Unit | Domain logic, workflow policies, LangGraph nodes with **mocked ports** |
| Integration | Adapter contract tests against **fakes**; optional tagged suites against real sandboxes |
| End-to-end | Full case flows with fake adapters and in-memory or test-container persistence |

---

## Mock ports by default

- **All ports and adapters are mocked or faked** in unit tests and default CI pipelines.
- Real external calls require an explicit test marker (e.g. `@pytest.mark.external`) and credentials
  from CI secrets—not from the repo.

Every new port in `src/ports/` must land with a matching fake in `tests/fakes/` before merge.

---

## Delivery gates

Before merge, changes must pass repository verification:

- Lint and format (`ruff`)
- Typecheck (when configured)
- Unit and integration tests
- No secrets in diff

---

## References

- [repo-layout.md](repo-layout.md)
- [integrations.md](integrations.md)
- [`AGENTS.md`](../../AGENTS.md) — development-agent verification checklist (not runtime product)
