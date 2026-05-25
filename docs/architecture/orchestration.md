# Orchestration (two-tier)

**Parent:** [OVERVIEW.md](OVERVIEW.md)

---

## Why two tiers

Business **case lifecycle** (sub-DAGs, approvals, compensations, SLAs) is separated from **in-step
agent reasoning** (tool use, reflection, retries). Collapsing both into one graph creates unmaintainable
coupling and weak audit boundaries.

| Tier | Responsibility | Technology |
|---|---|---|
| **1 — Workflow** | Sub-DAGs, HITL gates, case state, timeouts | Temporal — see [ADR-001](../adr/ADR-001-workflow-orchestration.md) |
| **2 — Agent reasoning** | Business-flow Agents orchestrating Reasoning-flow Skills | LangGraph / LangChain |

---

## Tier 1 — Workflow-level (Temporal)

- Models the end-to-end **case** as a graph of steps (sub-DAGs).
- Triggers **Business-flow Agents** as Temporal activities.
- Owns **HITL gates**: In-Agent HITL decisions (sign-off, mid-flow review, routing confirmation) before side effects — placement TBD per [ADR-006](../adr/ADR-006-in-agent-hitl-placement.md).
- Emits audit events for state transitions.

**Decision:** [ADR-001: Temporal](../adr/ADR-001-workflow-orchestration.md) for durable workflow execution.

---

## Tier 2 — Agent reasoning (LangGraph)

Tier 2 has two levels within LangGraph:

### Business-flow Agents

- One LangGraph **graph** per business flow (Intake, Response, Route to SME, Quality Assurance, Insight).
- Triggered by Temporal as an activity; returns a typed result back to Tier 1.
- Orchestrates Reasoning-flow Skills internally — selecting and sequencing the subset needed for that flow.
- Owns the fan-out of parallel work (e.g. processing multiple documents) via LangGraph `Send`.
- Checkpoints stored through the persistence abstraction ([persistence.md](persistence.md)).

### Reasoning-flow Skills

- Composable **subgraphs** with a defined interface: typed input/output and declared port dependencies.
- Available skills: Ingest, Parse & Validate, Enrichment, Elicitation, Q&A, Proposal.
- Not a fixed pipeline — each Business-flow Agent assembles the skills it needs.
- Handle tool calls to [integrations.md](integrations.md) via ports.
- Use **In-Skill HITL**: interrupts, resume with human-provided input, per-subgraph checkpointing.

---

## Execution chain

```
Temporal Workflow
  └── Business-flow Agent (LangGraph graph)
        ├── Skill A (subgraph)  ─→ Port → Adapter → External system
        ├── Skill B (subgraph)  ─→ Port → Adapter → External system
        └── [parallel via Send: Skill A × N documents]
```

---

## Interaction rules

1. Temporal **starts** and **awaits** Business-flow Agents; it does not embed LLM prompts in DAG definitions.
2. Business-flow Agents **compose** Skills; they do not re-implement skill logic inline.
3. Skills **do not** own case lifecycle or final sign-off policy — those stay in Tier 1.
4. Human approval that commits external changes must be recorded in **Tier 1** state — In-Agent HITL
   placement is resolved in [ADR-006](../adr/ADR-006-in-agent-hitl-placement.md).

---

## References

- [product-and-hitl.md](product-and-hitl.md)
- [persistence.md](persistence.md)
- [ADR-001](../adr/ADR-001-workflow-orchestration.md)
