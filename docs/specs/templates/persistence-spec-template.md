# Persistence Spec: [Skill or Agent Name]

> Filename convention: `persistence/<skill-or-agent-slug>-persistence-spec.md`
> One spec per Skill or Agent that uses checkpointing.
> Application code accesses storage only through `src/persistence/` — never via embedded SDK calls.
> For examples of how to fill in each section, see [TEMPLATE-README.md](TEMPLATE-README.md).

---

## Purpose

> One paragraph: what state this component needs to persist, why durability is required
> (HITL pause/resume, fault recovery, audit), and which Skill or Agent owns this state.

---

## State schema

> Describe the state TypedDict that LangGraph will checkpoint.

```python
from typing import TypedDict, Annotated
import operator
from src.domain import <DomainType>

class <SkillName>State(TypedDict):
    field_name: <DomainType>          # description
    optional_field: <DomainType> | None
    # For parallel fan-out results use a reducer:
    # results: Annotated[list[<DomainType>], operator.add]
```

| Field | Type | Description | Required |
|---|---|---|---|
| `field_name` | `<DomainType>` | | Yes |

---

## Checkpoint lifecycle

> When is state checkpointed, resumed, and cleared?

| Event | Action |
|---|---|
| Skill node completes | LangGraph auto-checkpoints via `src/persistence/` |
| HITL interrupt | State frozen; resume token handed to Temporal |
| Temporal resumes skill | `graph.invoke(Command(resume=human_decision))` restores state |
| Skill completes | Parent Agent reads output slot; skill-local state discarded |

---

## Backend

> State the persistence backend per environment. No direct backend SDK calls in skill code.

| Environment | Backend | Configured by |
|---|---|---|
| AWS PROD | DynamoDB | `src/persistence/` abstraction |
| OpenShift DEV/TEST | PostgreSQL | `src/persistence/` abstraction |

---

## Isolation requirements

> Skills always use skill-local state. Document any isolation constraints.

- This Skill's checkpoint is **isolated** from the parent Business-flow Agent state.
- Parallel `Send` instances each hold an **independent** checkpoint.
- [Any additional constraints]

---

## Open questions

- [ ] [unresolved question]

---

## References

- [persistence.md](../../architecture/persistence.md)
- [agents.md](../../architecture/agents.md)
- [ADR-002](../../adr/ADR-002-openshift-persistence.md)
- [Skill spec]()
