# Data and contracts

**Parent:** [OVERVIEW.md](OVERVIEW.md)

---

## Public API

- Style: **REST** over HTTPS.
- Versioning and idempotency: **[ADR-003](../adr/ADR-003-api-versioning-idempotency.md)**.
- Handlers validate at the HTTP boundary, then pass **domain types** inward.

---

## Internal contracts

- Shared payloads between API, workflow, agents, and adapters use **Pydantic v2** models in
  `src/domain/` ([ADR-004](../adr/ADR-004-domain-contracts-pydantic.md)).
- Ports accept and return domain types, not vendor JSON.
- Proposal and sign-off structures for HITL are explicit domain models with stable field names for
  audit serialization.

---

## Systems of record

- **Salesforce** and **DataHub** are primary SOR/integration targets ([integrations.md](integrations.md)).
- Field definitions and entity relationships come from **SOR metadata APIs**.
- Writes to SORs occur only after workflow **sign-off** unless an ADR defines an exception.

---

## References

- [product-and-hitl.md](product-and-hitl.md)
- [ADR-003](../adr/ADR-003-api-versioning-idempotency.md)
- [ADR-004](../adr/ADR-004-domain-contracts-pydantic.md)
