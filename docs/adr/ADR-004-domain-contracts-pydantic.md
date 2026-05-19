# ADR-004: Domain contracts with Pydantic

---

## Metadata

| Field | Value |
|---|---|
| Status | `accepted` |
| Date | 2026-05-18 |
| Deciders | Architecture baseline |
| Supersedes | — |
| Superseded by | — |

---

## Context

Data crosses several boundaries: REST API, Temporal workflows, LangGraph agent state, and external
adapters (Salesforce, DataHub, AI Gateway). Ad hoc `dict` payloads cause schema drift, weak
validation, and poor agent-generated code quality.

---

## Decision drivers

- Runtime validation at boundaries
- Shared types between API, workflow, agents, and tests
- Good Python ecosystem fit (already using Python 3.11+)
- JSON serialization for audit and persistence
- Clear mapping layer in adapters (vendor DTO → domain)

---

## Options considered

### Option A: Pydantic v2 models in `src/domain/`

**Description:** All cross-boundary payloads are Pydantic `BaseModel` (or `RootModel`) types.

**Pros:**
- Validation, serialization, and schema export (JSON Schema / OpenAPI)
- Familiar to Python agents and developers

**Cons:**
- Models must remain backward compatible or versioned explicitly

---

### Option B: dataclasses + manual validation

**Description:** Standard library only.

**Pros:**
- Fewer dependencies

**Cons:**
- No single schema source; more boilerplate; weaker agent ergonomics

---

### Option C: OpenAPI-first code generation

**Description:** Generate models from OpenAPI spec.

**Pros:**
- API contract is source of truth

**Cons:**
- Harder for agent-internal types not exposed on API; two sources of truth without discipline

---

## Decision

**We choose Option A — Pydantic v2** for all domain contracts in `src/domain/`, with adapters
responsible for mapping vendor types to and from these models.

### Conventions

| Topic | Rule |
|---|---|
| Location | `src/domain/` — subpackages by bounded context (`case`, `proposal`, `sor`, …) |
| Naming | `PascalCase` models; `snake_case` fields |
| Immutability | Prefer frozen models for values passed across tiers where practical |
| Errors | Shared `DomainError` hierarchy; map to HTTP in API layer only |
| Versioning | Add optional fields for non-breaking changes; new model version suffix for breaking (`ProposalV2`) |
| Ports | Method signatures use domain types, not `dict` or vendor SDK types |

---

## Consequences

### Positive

- One validated shape for proposals, HITL decisions, and adapter outputs
- Fakes in tests use the same types as production
- OpenAPI can be generated from route models aligned with domain

### Negative / Trade-offs

- Discipline required to avoid importing adapter types into `domain`
- Large graphs may need selective serialization for LangGraph state (exclude heavy blobs by reference)

### Neutral

- REST idempotency remains a transport concern ([ADR-003](ADR-003-api-versioning-idempotency.md))

---

## Implementation notes

- Pin `pydantic>=2` in project dependencies when `pyproject.toml` is extended for the application.
- Add `model_config = ConfigDict(extra="forbid")` on external-facing request models.
- Store document content in object storage; domain models carry **references** (URI, checksum), not raw files.

**Related architecture:** [data-and-contracts.md](../architecture/data-and-contracts.md), [integrations.md](../architecture/integrations.md)
