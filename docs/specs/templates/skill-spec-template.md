# Skill Spec: [Skill Name]

> Filename convention: `skills/<skill-slug>-spec.md`
> One spec per Skill. Fill in every section; remove instructional notes before review.
> For examples of how to fill in each section, see [TEMPLATE-README.md](TEMPLATE-README.md).

---

## Purpose

> One paragraph: what business problem this Skill solves and what triggers it.

---

## Interface

### Input

> Pydantic model from `src/domain/`. Link to the domain contract spec and list fields.

| Field | Type | Required | Description |
|---|---|---|---|
| `field_name` | `FieldType` | Yes | |

Domain contract spec: [link]()

### Output

> Pydantic model from `src/domain/`. Link to the domain contract spec and list fields.

| Field | Type | Required | Description |
|---|---|---|---|
| `field_name` | `FieldType` | Yes | |

Domain contract spec: [link]()

---

## Dependencies

| Component | Name | Spec |
|---|---|---|
| Tool | `<ToolName>` | [link to tool spec]() |
| Persistence | LangGraph checkpoint | [persistence.md](../../architecture/persistence.md) |
| Observability | LangSmith trace | [observability.md](../../architecture/observability.md) |

---

## Internal flow

> Describe the reasoning steps as a numbered list or Mermaid diagram.
> Include where Tools are invoked and which Tool is called at each step.
> Include where HITL interrupts occur if applicable.
> See [TEMPLATE-README.md](TEMPLATE-README.md#skill-spec-template--internal-flow-diagram) for a diagram example.

---

## HITL

> Does this Skill interrupt for human input? If yes, describe:
> - Trigger condition
> - Payload sent to human (`ElicitationRequest` fields)
> - How human response is used to resume

None / [describe interrupt]

---

## Error handling

> List the domain errors this Skill may surface and the expected caller reaction.
> Skills do not retry — errors propagate to the workflow orchestration layer.

| Error | Condition | Caller reaction |
|---|---|---|
| `<DomainError>` | <when> | <what Temporal does> |

---

## Observability

> LangSmith trace tags required. Add any skill-specific structured log events beyond the defaults.

- Tags: `case_id`, `workflow_id`, `step_id` (mandatory on all runs)
- Additional events: [list any]

---

## Open questions

- [ ] [unresolved question]

---

## References

- [agents.md](../../architecture/agents.md)
- [Domain Contract spec](domain-contract-spec-template.md)
- [Tool specs]()
