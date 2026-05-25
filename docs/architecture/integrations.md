# External integrations

**Parent:** [OVERVIEW.md](OVERVIEW.md)

---

## Port–adapter pattern

Every external system is accessed through a **port** (interface in `src/ports/`) and a concrete
**adapter** (in `src/adapters/`). Domain code depends on ports only.

The full call chain through the system is:

```
LangGraph Agent → Tool function → Port → Adapter → External system / SDK
```

| Dependency | Purpose | Adapter location (planned) |
|---|---|---|
| **Okta** | Identity / M2M tokens | `src/adapters/okta/` |
| **AI Gateway** | LLM inference | `src/adapters/ai_gateway/llm.py` |
| **AI Gateway** | KB / RAG | `src/adapters/ai_gateway/rag.py` |
| **SOR** | System of record | `src/adapters/sor/` |
| **SOR metadata** | Entity and field schema catalog | `src/adapters/sor_metadata/` |
| **Email** | Document intake | `src/adapters/email/` |
| **External storage** | Document intake | `src/adapters/storage/` |

---

## Port–adapter design principles

### Principles

**P1 — Tools are the agent-to-port bridge**
Agents call external services using LangGraph Tools only. Tools call ports; ports have adapters.
*(→ [Rationale P1](#rationale))*

---

**P2 — Behavioral contracts in ports; data contracts in `src/domain/`**
Ports define behavioral contracts: what operations exist, their inputs, outputs, and errors.
Shared data contracts — Pydantic models, enums, value objects, and typed errors — belong in
`src/domain/` ([ADR-004](../adr/ADR-004-domain-contracts-pydantic.md)).

*Exception:* Highly specialized schemas for raw serialization/deserialization (e.g. matching a
legacy API payload exactly) live in `src/adapters/`, not `src/ports/`. The port's method
signatures remain domain-typed. The adapter translates internally: raw DTO → domain model before
passing anything inward.
*(→ [Rationale P2](#rationale))*

---

**P3 — Port interfaces express domain intent**
Port method signatures express what the domain needs, not how the vendor implements it.
Technology-specific parameters must not appear on a port interface.
*(→ [Rationale P3](#rationale))*

---

**P4 — Adapters satisfy ports through structural typing**
Adapters satisfy port interfaces through structural typing — no explicit inheritance needed. The
dependency flows one way: agents, tools, and adapters all depend on ports; nothing depends on
adapters except the composition root.
*(→ [Rationale P4](#rationale))*

---

**P5 — Every port ships with a fake adapter**
Every port ships with a fake adapter in `tests/fakes/` as a shared, reusable test artifact.
The fake must exist before the production adapter is merged. Mocks are a supplementary tool,
not a substitute for the fake.
*(→ [Rationale P5](#rationale))*

---

**P6 Adapters report failures; workflow orchestration owns the reaction**
Adapters must convert transient vendor- or technology-specific failures into typed domain errors/exceptions defined in `src/domain`. Adapters never choose how to react to a failure: no loop, sleep, or retry, or else. All failure handling policies—including retries, exponential backoffs, human-in-the-loop interventions, or graph termination—belong exclusively in the workflow orchestration layer.
*(→ [Rationale P6](#rationale))*

---

### Rationale

**P1** — The word *only* has enforcement value: the complete set of external capabilities
available to an agent is fully visible by reading its tool list at the graph definition. There
is no hidden path to the outside world.

**P2** — If a data type lives inside a port, every other port that needs the same concept must
either duplicate it or import from a sibling port. Both choices weaken the architecture:
duplication creates drift, while sibling-port imports create hidden coupling. Shared domain
concepts belong in `src/domain/`, which may be imported freely by ports, adapters, tools,
agents, and application services. Ports should depend on the domain layer, not on each other.
Port-specific DTOs may stay with the adapter.

**P3** — Technology-specific parameters include connection details, protocol options, pagination
cursors, retry hints, serialization flags, and anything else that belongs in a vendor SDK, API,
or framework reference rather than a business requirements document. The adapter decides how to
satisfy the domain intent using whatever infrastructure controls are appropriate. A useful test:
if a parameter would not appear in a business requirements document, it does not belong on a
port. When a vendor changes — or is swapped entirely — the port interface should not need to
change.

**P4** — Python's `typing.Protocol` is the preferred mechanism: define the port as a `Protocol`,
and any adapter with matching methods satisfies it structurally, without subclassing. The type
checker enforces conformance at usage sites. The composition root is the only place where a
concrete adapter is instantiated and injected — agent and tool modules never import from
`src/adapters/` directly.

**P5** — A fake is a working in-memory implementation of the port's Protocol — it stores state,
returns plausible responses, and can be shared across the entire test suite without per-test
configuration. Writing a fake forces the interface to be fully implemented; if the fake is hard
to write, the port design is probably wrong. Mocks (`MagicMock`, `pytest-mock`) are used
selectively where interaction verification is needed — asserting that a method was called, how
many times, or with what arguments — or to inject failures a fake cannot easily simulate. In
practice: fakes cover the majority of tests; mocks are pulled in only when the fake is
insufficient for a specific test.

**P6** — Adapters should report what went wrong in domain terms so callers can decide policy.
Embedding retry loops inside adapters duplicates logic across integrations, hides failure modes
from workflow observability, and makes idempotency harder to enforce at the case level.
Transient vs permanent failure classification stays at the adapter boundary; whether and how to
retry stays with workflow policies that understand case state and side effects.

---

## References

- [authentication.md](authentication.md)
- [data-and-contracts.md](data-and-contracts.md)
- [quality-and-delivery.md](quality-and-delivery.md)
- [repo-layout.md](repo-layout.md)
