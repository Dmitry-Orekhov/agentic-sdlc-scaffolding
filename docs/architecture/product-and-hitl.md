# Product model and human-in-the-loop

**Parent:** [OVERVIEW.md](OVERVIEW.md)

---

## Augmented Intelligence

The human–agent relationship is **Augmented Intelligence**, not full automation:

- The **human** holds tacit and tribal knowledge, applies experience and judgment, and makes the
  **final decision**.
- **Agents** execute supportive work: intake, parsing, completeness checks, calls to external tools,
  answering questions, and—when needed—**elicitation** requests back to the human.

Agents produce **proposals**; humans **sign off**, reject, or request revision before outcomes are
finalized.

---

## HITL on the critical path

Human-in-the-loop is a **core** capability, not a failure or edge-case handler.

Implementation expectations:

- Workflow states must include explicit **human tasks** (questions, reviews, approvals).
- Pausing, resuming, and timeout behavior must be **durable** (see [persistence.md](persistence.md)).
- Audit must record **what** was proposed, **who** decided, and **when** (see
  [observability.md](observability.md)).
- Two HITL patterns are defined: **In-Skill HITL** (scoped to a Reasoning-flow Skill instance)
  and **In-Agent HITL** (scoped to a Business-flow Agent — sign-off, mid-flow review, routing
  confirmation). See [agents.md](agents.md) and [ADR-006](../adr/ADR-006-in-agent-hitl-placement.md).
- LangGraph **interrupts** and workflow-level **signals** are first-class mechanisms — see
  [orchestration.md](orchestration.md).

---

## Agent taxonomy

### Business-flow Agents

Each Business-flow Agent is a **LangGraph graph** mapped to one top-level business flow.
They are triggered by the Temporal orchestrator as activities and compose Reasoning-flow
Skills internally.

| Agent | Responsibility |
|---|---|
| **Intake** | Accept and route incoming documents from email and external storage |
| **Response** | Produce a human-reviewed response or outcome for a case |
| **Route to SME** | Determine and assign the appropriate subject-matter expert |
| **Quality Assurance** | Validate case handling quality against defined criteria |
| **Insight** | Aggregate analytics to identify improvements across the whole flow |

---

### Reasoning-flow Skills

Skills are **composable subgraphs** with a defined interface (typed input/output, declared
port dependencies). Business-flow Agents are not required to use all skills — each flow
assembles the subset it needs. Skills are not a fixed pipeline.

| Skill | Responsibility |
|---|---|
| **Ingest** | Accept and normalise raw input (documents, payloads) |
| **Parse & Validate** | Extract structure; check completeness against rules and SOR metadata |
| **Enrichment** | Call external tools and SORs to add missing or supporting data |
| **Elicitation** | Interrupt flow and request human input when data is missing or ambiguous |
| **Q&A** | Answer bounded questions using KB/RAG and case context |
| **Proposal** | Package reasoning output into a structured recommendation for human sign-off |

---

## References

- [orchestration.md](orchestration.md) — where HITL gates live in the two-tier model
- [data-and-contracts.md](data-and-contracts.md) — proposal and decision payloads
