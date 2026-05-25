# ADR-005: Reasoning-flow Skills as the composition unit for Business-flow Agents

---

## Metadata

| Field | Value |
|---|---|
| Status | `accepted` |
| Date | 2026-05-23 |
| Deciders | @Dmitry-Orekhov |
| Supersedes | — |
| Superseded by | — |

---

## Context

Business-flow Agents (Intake, Response, Route to SME, Quality Assurance, Insight) share
overlapping reasoning capabilities: parsing documents, enriching with SOR data, asking the
KB, eliciting human input, and assembling proposals. Without a defined composition unit,
each Business-flow Agent re-implements these capabilities in isolation, causing duplication,
inconsistent HITL behaviour, and untestable logic.

A composition unit is needed that:
- Can be reused across multiple Business-flow Agents
- Supports parallel fan-out (e.g. processing N documents independently)
- Has isolated checkpointing and HITL interrupt scope
- Is independently testable with fake adapters

---

## Decision Drivers

- Reuse of reasoning logic across Business-flow Agents without copy-paste
- Per-unit checkpoint and resume granularity (critical for long-running parallel tasks)
- HITL interrupt scope must be narrow enough to pause one task without affecting others
- Skills must be independently testable in isolation from the Business-flow Agent
- No new runtime infrastructure — must stay within the existing LangGraph stack

---

## Options Considered

### Option A: Plain nodes in the Business-flow Agent graph

**Description**: Reasoning steps (parse, enrich, etc.) are implemented as plain LangGraph
nodes directly inside each Business-flow Agent graph. No shared subgraph structure.

**Pros**:
- Simplest to implement initially
- No state mapping overhead — all nodes share the parent graph's state

**Cons**:
- No reuse — each Business-flow Agent re-implements the same logic
- Parallel fan-out with `Send` is awkward: nodes share state, making per-document isolation
  hard to achieve cleanly
- HITL interrupt pauses the entire parent graph, not just the affected document task
- Checkpointing is at the parent graph level — cannot resume a single failed document task
- Testing requires standing up the full Business-flow Agent graph

---

### Option B: Skills as composable LangGraph subgraphs

**Description**: Each reasoning capability (Ingest, Parse & Validate, Enrichment,
Elicitation, Q&A, Proposal) is implemented as a compiled LangGraph subgraph — a **skill**
— with a defined interface: typed input, typed output, and declared port dependencies.
Skills live in a shared library (`src/agents/skills/`). Business-flow Agents compose from
this library, selecting and sequencing only the skills they need.

**Pros**:
- Full reuse across Business-flow Agents — one implementation, many consumers
- `Send`-based parallel fan-out works naturally: each document gets its own skill instance
  with isolated state and checkpoint
- HITL interrupts are scoped to the skill instance — one document's elicitation does not
  block others
- Skills are independently testable with fake port adapters
- Skill interface is a stable versioning boundary

**Cons**:
- State mapping required between parent graph and skill subgraph (input/output translation)
- Slightly more boilerplate per composition compared to plain nodes
- Shared state schema discipline required — skills must not make assumptions about parent
  state beyond their declared input contract

---

### Option C: Skills as separate agent microservices

**Description**: Each skill is deployed as an independent service invoked over the network
by Business-flow Agents.

**Pros**:
- Maximum isolation — independent deployability and scaling per skill

**Cons**:
- Significant operational overhead for fine-grained reasoning steps
- Network latency and failure modes inside what is logically one reasoning flow
- No benefit over subgraphs for the current deployment scale
- Contradicts the monorepo, single-deployment architecture

---

## Decision

**We choose Option B — Skills as composable LangGraph subgraphs.**

Subgraphs provide the necessary isolation for parallel fan-out and per-task HITL without
introducing new runtime infrastructure. The state mapping overhead is manageable and is
outweighed by the reuse and testability benefits. Option A forecloses parallel processing
and per-task resume. Option C is premature complexity.

---

## Consequences

### Positive
- Reasoning logic is implemented once and reused across all Business-flow Agents
- Parallel document processing is cleanly supported via `Send` with per-instance isolation
- HITL interrupts and checkpoints are scoped to the skill, not the whole Business-flow Agent
- Skills are the unit of reuse, independent testing, and versioning

### Negative / Trade-offs
- Each skill composition requires explicit input/output state mapping at the parent boundary
- The shared skill library (`src/agents/skills/`) becomes a coordination point — changes to
  a skill interface affect all consumers

### Neutral
- Skills follow the same port-injection pattern as Business-flow Agents — no new conventions
  needed for tool access

---

## Open Questions

- [x] **State strategy**: resolved — see State Boundary below.
- [ ] **Skill versioning**: how are breaking interface changes to a skill propagated to
  consuming Business-flow Agents? Needs a convention (semver, deprecation period, ADR).
- [ ] **Skill registry**: is `src/agents/skills/` self-describing enough, or is a runtime
  registry needed for dynamic composition?

---

## State Boundary

**Decision: skills always use skill-local state.**

This applies uniformly to both sequential and parallel skills — no exceptions for "simple"
cases. Consistency eliminates a class of bugs where a skill accidentally reads or writes
parent state beyond its declared contract.

**How the Business-flow Agent tracks execution:**

- **Sequential skills** — the skill returns a typed output; the parent maps it into a
  dedicated field in its own state before invoking the next skill.
- **Parallel fan-out via `Send`** — the parent state declares a reducer field
  (`Annotated[list[SkillOutput], operator.add]`); each skill instance appends its result;
  the parent graph continues once all instances complete.

```python
# Parent state — output slots declared per skill
class BusinessFlowState(TypedDict):
    documents: list[Document]
    parsed_results: Annotated[list[ParseResult], operator.add]  # parallel reducer

# Fan-out
def dispatch_parse(state: BusinessFlowState):
    return [Send("parse_validate", {"document": doc}) for doc in state.documents]
```

The parent graph observes skill outputs only through these declared output slots — never
through skill-internal state. This is the enforced contract between Business-flow Agent
and its skills.

---

## Implementation Notes

- Skills live in `src/agents/skills/<skill-name>/`
- Each skill exposes a `build_<skill>_graph(ports...) -> CompiledGraph` factory function
- Skill input/output types are Pydantic domain models defined in `src/domain/`
- Every skill ships with a fake-adapter test in `tests/unit/skills/`
- See [orchestration.md](../architecture/orchestration.md) for the execution chain
- See [product-and-hitl.md](../architecture/product-and-hitl.md) for the full skill taxonomy
