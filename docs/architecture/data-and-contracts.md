# Data and contracts

**Parent:** [OVERVIEW.md](OVERVIEW.md)

---

## Public API

- Style: **REST** over HTTPS.
- Versioning and idempotency: **[ADR-003](../adr/ADR-003-api-versioning-idempotency.md)**.
- Handlers validate at the HTTP boundary, then pass **domain types** inward to workflow and
  services — a parallel entry path to the agent tool chain described in
  [integrations.md](integrations.md).

---

## Internal contracts

- Shared payloads between API, workflow, agents, and adapters use **Pydantic v2** models in
  `src/domain/` ([ADR-004](../adr/ADR-004-domain-contracts-pydantic.md)).
- Ports accept and return domain types, not vendor JSON; adapters map vendor DTOs at the boundary
  ([integrations.md](integrations.md) P2).
- Adapters surface failures as typed domain errors defined in `src/domain/`; retry, backoff, and
  HITL reaction policies belong in workflow orchestration ([integrations.md](integrations.md) P6).
- Proposal and sign-off structures for HITL are explicit domain models with stable field names for
  audit serialization.

---

## Systems of record

- External integrations are defined in [integrations.md](integrations.md).
- **SOR** is the authoritative store for case data: reads for enrichment anytime; writes for
  finalized outcomes only after workflow **sign-off** unless an ADR defines an exception.
- **SOR metadata** is the read-only schema catalog for entity and field definitions.
- Load field definitions and entity relationships from **SOR metadata APIs** at runtime rather than
  hard-coding schema assumptions.
- Cache metadata with TTL where appropriate; invalidate on version changes reported by SOR metadata
  endpoints.

---

## References

- [integrations.md](integrations.md)
- [product-and-hitl.md](product-and-hitl.md)
- [ADR-003](../adr/ADR-003-api-versioning-idempotency.md)
- [ADR-004](../adr/ADR-004-domain-contracts-pydantic.md)
