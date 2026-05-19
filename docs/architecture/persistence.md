# Agent and workflow state persistence

**Parent:** [OVERVIEW.md](OVERVIEW.md)

---

## Requirements

Durable storage is required for:

- Temporal workflow history and HITL wait states
- LangGraph **checkpoints** and resume tokens
- Case correlation IDs and audit pointers

Application code accesses storage only through `src/persistence/`, not via embedded SDK calls in
graphs or workflow definitions.

---

## Backends by environment

| Environment | Backend | ADR |
|---|---|---|
| AWS PROD | **Amazon DynamoDB** | Baseline; table design in implementation |
| OpenShift DEV/TEST | **PostgreSQL** | [ADR-002](../adr/ADR-002-openshift-persistence.md) |

The persistence module exposes a single interface; environment configuration selects the backend
implementation at startup.

---

## LangGraph checkpointing

- On OpenShift: use PostgreSQL-backed checkpointer compatible with LangGraph.
- On AWS PROD: use the DynamoDB-backed implementation selected in [ADR-002](../adr/ADR-002-openshift-persistence.md).

---

## References

- [orchestration.md](orchestration.md)
- [runtime-topology.md](runtime-topology.md)
- [ADR-002](../adr/ADR-002-openshift-persistence.md)
