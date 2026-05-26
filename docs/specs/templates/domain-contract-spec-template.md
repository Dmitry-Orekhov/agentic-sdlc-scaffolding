# Domain Contract Spec: [Model Name]

> Filename convention: `domain/<model-name-as-slug>-spec.md` (e.g. `ParseResult` → `parse-result-spec.md`)
> One spec per Pydantic model or related group of models (e.g. input/output pair for one Skill).
> See ADR-004 for the domain contracts decision.
> For examples of how to fill in each section, see [TEMPLATE-README.md](TEMPLATE-README.md).

---

## Purpose

> One paragraph: what concept this model represents and why it belongs in `src/domain/`
> rather than in a port or adapter.

---

## Model definition

| Field | Type | Required | Description | Constraints |
|---|---|---|---|---|
| `field_name` | `FieldType` | Yes | | |
| `optional_field` | `FieldType \| None` | No | | |

> Example (non-normative):

```python
# src/domain/<module>.py
from pydantic import BaseModel, Field

class <ModelName>(BaseModel):
    field_name: FieldType = Field(..., description="...")
    optional_field: FieldType | None = Field(None, description="...")
```

---

## Validation rules

> List any cross-field validators, field constraints, or invariants that Pydantic enforces.
> See [TEMPLATE-README.md — Validation rules examples](TEMPLATE-README.md#domain-contract-spec-template--validation-rules).

1. [rule]

---

## Serialization notes

> Requirements affecting how this model is written to or read from external systems
> (audit logs, REST responses, persistence, inter-service payloads).
> Omit this section if the model is internal-only with no special requirements.
> See [TEMPLATE-README.md — Serialization notes examples](TEMPLATE-README.md#domain-contract-spec-template--serialization-notes).

1. [serialization note]

---

## Open questions

- [ ] [unresolved question]

---

## References

- [data-and-contracts.md](../../architecture/data-and-contracts.md)
- [ADR-004](../../adr/ADR-004-domain-contracts-pydantic.md)
- [Skill spec]()
