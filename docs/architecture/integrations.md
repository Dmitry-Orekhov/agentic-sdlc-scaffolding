# External integrations

**Parent:** [OVERVIEW.md](OVERVIEW.md)

---

## Port–adapter pattern

Every external system is accessed through a **port** (interface in `src/ports/`) and a concrete
**adapter** (in `src/adapters/`). Domain code depends on ports only.

| Dependency | Purpose | Adapter location (planned) |
|---|---|---|
| **Okta** | Identity / M2M tokens | `src/adapters/okta/` |
| **AI Gateway** | LLM inference | `src/adapters/ai_gateway/llm.py` |
| **AI Gateway** | KB / RAG | `src/adapters/ai_gateway/rag.py` |
| **Salesforce** | System of record | `src/adapters/salesforce/` |
| **DataHub** | SOR metadata / Redshift-backed API | `src/adapters/datahub/` |
| **Email** | Document intake | `src/adapters/email/` |
| **External storage** | Document intake | `src/adapters/storage/` |

---

## SOR metadata

Metadata describing SOR entities and fields is available via **API**. Agents and services must load
metadata at runtime rather than hard-coding schema assumptions. Cache with TTL where appropriate;
invalidate on version changes from DataHub or Salesforce metadata endpoints.

---

## Adapter implementation rules

1. Map vendor DTOs to **domain models** at the adapter boundary ([ADR-004](../adr/ADR-004-domain-contracts-pydantic.md)).
2. Surface transient failures as typed errors; let workflow policies handle retries.
3. Never log credentials, full tokens, or unredacted PII.
4. Ship a **fake adapter** for every port before merging production adapter code.

---

## References

- [authentication.md](authentication.md)
- [data-and-contracts.md](data-and-contracts.md)
- [quality-and-delivery.md](quality-and-delivery.md)
- [ADR-004](../adr/ADR-004-domain-contracts-pydantic.md)
