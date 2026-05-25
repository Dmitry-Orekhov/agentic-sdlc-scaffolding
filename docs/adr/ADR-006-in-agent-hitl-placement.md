# ADR-006: In-Agent HITL placement

---

## Metadata

| Field | Value |
|---|---|
| Status | `proposed` |
| Date | 2026-05-23 |
| Deciders | @Dmitry-Orekhov |
| Supersedes | — |
| Superseded by | — |

---

## Context

The architecture defines two HITL patterns:

- **In-Skill HITL** — a Reasoning-flow Skill (e.g. Elicitation) interrupts mid-reasoning to
  request missing or ambiguous data from a human. Scoped to the skill instance.
- **In-Agent HITL** — the Business-flow Agent requires a human decision at the agent graph
  level. Scoped to the Business-flow Agent. Use cases include:
  - **Final sign-off** — human approves the full proposal before Temporal commits side effects
  - **Mid-flow review** — human reviews intermediate state before the agent continues
  - **Routing confirmation** — human confirms an assignment or routing decision

The current architecture documents do not define where In-Agent HITL lives. Two options
exist: as a **Temporal HITL gate** wrapping the Business-flow Agent activity, or as an
**in-agent node inside the Business-flow Agent graph**.

This decision has direct consequences for audit boundaries, serialization contracts, and the
separation of reasoning from approval policy.

---

## Decision Drivers

- Human decisions that commit external state changes must be auditable at the case level
- Approval policy (who can approve, timeout, escalation) must not be embedded in agent reasoning logic
- The payload presented to the human may benefit from the full LangGraph reasoning context
- Consistency with the existing rule: "LangGraph graphs do not own case lifecycle or final sign-off policy"
- Minimise the number of distinct HITL mechanisms to maintain

---

## Options Considered

### Option A: In-Agent HITL as a Temporal gate

**Description**: The Business-flow Agent produces a typed output (e.g. `Proposal`,
`RoutingDecision`) and returns it to Temporal. Temporal pauses the workflow, creates a
human task, and waits for a `HumanDecision` signal before continuing to the next workflow
step (e.g. SOR write, next Business-flow Agent).

```
Temporal Workflow
  └── Business-flow Agent activity → typed output (returned to Temporal)
  └── [Temporal HITL gate] Human reviews output → approve / reject / revise
  └── Next activity — only if approved
```

**Pros**:
- Consistent with the existing architectural rule: approval policy stays in Tier 1
- Temporal owns the full audit record: output payload, human actor, decision, timestamp —
  all in one place alongside case state and downstream side effects
- Business-flow Agent is a pure reasoning unit — it produces an output and exits cleanly
- Approval policy (timeout, escalation, who can approve) is configurable in Temporal without
  touching agent code
- Single HITL mechanism — Temporal signals — used for all agent-level human decisions
- Output serialization contract is explicit: a Pydantic domain model returned as an
  activity result, not embedded in graph state
- Applies uniformly to all In-Agent HITL use cases (sign-off, mid-flow review, routing)

**Cons**:
- The full LangGraph reasoning context (message history, intermediate tool outputs) is not
  directly available to the human reviewer — only what the output model carries
- Temporal must deserialize and display the output to the human task UI — requires a clean,
  stable serialization contract for every output type
- Revision requests require re-invoking the Business-flow Agent as a new activity, passing
  revision notes as input — slightly more orchestration boilerplate per use case

---

### Option B: In-Agent HITL as a node in the Business-flow Agent graph

**Description**: A dedicated HITL node (e.g. `sign_off`, `confirm_routing`) is added to
the Business-flow Agent graph at the appropriate point. It calls `interrupt(payload)`,
pauses the graph, and waits for a `HumanDecision` resume signal before the agent continues
or returns its result to Temporal.

```
Business-flow Agent (LangGraph graph)
  └── ... skills ...
  └── hitl_node → interrupt(payload) → [human reviews] → resume(HumanDecision)
  └── finalize node → return result to Temporal
```

**Pros**:
- Full LangGraph reasoning context (message history, intermediate outputs) is available at
  review time — agent can present a richer, contextualised summary to the human
- Revision requests can trigger a reasoning loop back into earlier skills without exiting
  the graph — tighter iteration between human and agent
- One fewer Temporal signal/activity type to define per use case

**Cons**:
- Violates the existing interaction rule: "LangGraph graphs do not own case lifecycle or
  final sign-off policy" — approval policy (timeout, escalation, who can approve) would
  need to live inside or alongside the agent graph
- Audit record is split: reasoning in LangSmith/LangGraph, human decision in Temporal —
  correlating them requires explicit cross-referencing
- Two distinct HITL mechanisms in the system: In-Skill interrupts and In-Agent interrupts —
  different resume paths, different checkpoint scopes, more surface area to maintain
- Business-flow Agent activity does not return until the human decides — long-running
  Temporal activity holding state for the duration of human review (minutes to hours)
- If the human requests revision requiring skill re-runs, the graph must checkpoint and
  restart subgraphs — complex to implement correctly

---

### Option C: In-Agent HITL as a dedicated Business-flow Agent

**Description**: Each In-Agent HITL use case is its own lightweight Business-flow Agent —
a LangGraph graph whose sole job is to present a payload and collect a human decision.
The preceding agent returns its output to Temporal; Temporal invokes the HITL Agent as the
next activity.

```
Temporal Workflow
  └── Business-flow Agent activity → typed output
  └── HITL Agent activity (e.g. SignOffAgent) → HumanDecision
  └── Next activity — only if approved
```

**Pros**:
- HITL logic is isolated and reusable across flows sharing the same approval pattern
- Business-flow reasoning agents remain pure reasoning units
- LangGraph context from the preceding agent can be passed as input if needed

**Cons**:
- Introduces agent types whose primary job is HITL coordination — closer to a Temporal
  activity wrapper than a reasoning agent, muddying the agent taxonomy
- Approval policy still risks leaking into the agent graph unless carefully constrained
- Additional complexity for something Temporal already handles natively

---

## Decision

> To be decided.

Key question: does the value of having LangGraph reasoning context available at review time
(Option B/C) outweigh the architectural cost of mixing approval policy into the agent layer
(Option B) or adding coordination-only agents (Option C)?

The existing interaction rules and audit requirements lean toward **Option A**, but the
revision-loop use case (human rejects → agent revises → human re-reviews) needs to be
explicitly addressed before the decision is finalised.

---

## Consequences

> To be filled once the decision is made.

---

## Open Questions

- [ ] Does the revision-loop use case (reject → revise → re-review) need to be tight
  (within one agent run) or is re-invoking the Business-flow Agent from Temporal acceptable?
- [ ] What data must output models carry to give the human reviewer sufficient context
  without embedding full message history?
- [ ] Is the long-running activity duration (Option B) a practical concern given Temporal's
  heartbeat and timeout configuration?

---

## References

- [orchestration.md](../architecture/orchestration.md) — interaction rules, HITL gates
- [product-and-hitl.md](../architecture/product-and-hitl.md) — HITL as a core capability
- [agents.md](../architecture/agents.md) — HITL mechanics at both levels
- [ADR-001](ADR-001-workflow-orchestration.md) — Temporal as workflow orchestrator
- [ADR-005](ADR-005-reasoning-flow-skills.md) — skill composition and state boundary
