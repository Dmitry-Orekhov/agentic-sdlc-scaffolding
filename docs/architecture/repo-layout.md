# Repository and code layout

**Parent:** [OVERVIEW.md](OVERVIEW.md)

---

## Single monorepo

All application code, adapters, workflow definitions, agent graphs, API, and tests live in **one
repository**. Shared domain types and ports prevent drift between agents and services.

---

## Planned top-level layout

```
src/
  api/              # Public REST API (FastAPI or equivalent)
  workflow/         # Temporal workflows and activities
  agents/           # LangGraph graph definitions and nodes
  domain/           # Pydantic models, enums, shared errors
  ports/            # Abstract interfaces (Protocols / ABCs)
  adapters/         # Port implementations (SOR, SOR metadata, AI Gateway, …)
  persistence/      # State store abstraction and backends
  intake/           # Email and external storage ingestion
  config/           # Settings loading per environment
  observability/    # Logging, tracing, audit helpers
tests/
  unit/
  integration/
  fakes/            # In-memory adapters for all ports
docs/
  architecture/     # This documentation set
  adr/              # Decision records
```

Paths are **prescriptive for greenfield code** until `src/` exists; new modules should follow this
map unless an ADR or feature architecture doc defines an exception.

---

## Module rules

| Rule | Detail |
|---|---|
| Dependency direction | `api` → `workflow` → `agents` → `ports` ← `adapters`; `domain` imported by all, imports nothing outward |
| No vendor SDKs in `domain` or `ports` | SDKs only in `adapters` |
| One port per external capability | e.g. `SorPort`, `SorMetadataPort`, `LlmPort`, `RagPort` |
| Fakes required | Every port has a `tests/fakes/` or `adapters/fake_*` implementation |

---

## References

- [integrations.md](integrations.md) — port–adapter detail
- [quality-and-delivery.md](quality-and-delivery.md) — testing layout
- [ADR-004](../adr/ADR-004-domain-contracts-pydantic.md)
