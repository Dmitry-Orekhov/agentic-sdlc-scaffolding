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
- LangGraph **interrupts** and workflow-level **signals** are first-class mechanisms—see
  [orchestration.md](orchestration.md).

---

## Typical agent responsibilities

| Area | Agent role |
|---|---|
| Intake | Accept documents from email and external storage |
| Parse & validate | Extract structure, check completeness against rules/metadata |
| Enrich | Call external tools and SORs for additional data |
| Q&A | Answer bounded questions using KB/RAG and case context |
| Elicitation | Request human input when data is missing or ambiguous |
| Proposal | Package recommendations for human sign-off |

---

## References

- [orchestration.md](orchestration.md) — where HITL gates live in the two-tier model
- [data-and-contracts.md](data-and-contracts.md) — proposal and decision payloads
