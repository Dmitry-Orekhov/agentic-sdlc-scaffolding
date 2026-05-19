# Orchestration (two-tier)

**Parent:** [OVERVIEW.md](OVERVIEW.md)

---

## Why two tiers

Business **case lifecycle** (sub-DAGs, approvals, compensations, SLAs) is separated from **in-step
agent reasoning** (tool use, reflection, retries). Collapsing both into one graph creates unmaintainable
coupling and weak audit boundaries.

| Tier | Responsibility | Technology |
|---|---|---|
| **1 — Workflow** | Sub-DAGs, HITL gates, case state, timeouts | See [ADR-001](../adr/ADR-001-workflow-orchestration.md) |
| **2 — Agent reasoning** | LangGraph graphs per workflow step | LangGraph / LangChain |

---

## Tier 1 — Workflow-level

- Models the end-to-end **case** as a graph of steps (sub-DAGs).
- Invokes Tier 2 agent graphs as **activities** or child workflows.
- Owns **HITL gates**: elicitation, review, sign-off before side effects.
- Emits audit events for state transitions.

**Decision:** [ADR-001: Temporal](../adr/ADR-001-workflow-orchestration.md) for durable workflow execution.

---

## Tier 2 — Agent reasoning (LangGraph)

- One LangGraph graph (or compiled subgraph) per agent **step type**.
- Handles tool calls to [integrations.md](integrations.md) via ports.
- Uses **native HITL** support: interrupts, resume with human-provided input, checkpointing.
- Checkpoints stored through the persistence abstraction ([persistence.md](persistence.md)).

---

## Interaction rules

1. Workflow code **starts** and **awaits** agent steps; it does not embed LLM prompts in DAG definitions.
2. LangGraph graphs **do not** own case lifecycle or final sign-off policy.
3. Human approval that commits external changes must be recorded in **Tier 1** state after Tier 2
   produces a proposal.

---

## References

- [product-and-hitl.md](product-and-hitl.md)
- [persistence.md](persistence.md)
- [ADR-001](../adr/ADR-001-workflow-orchestration.md)
