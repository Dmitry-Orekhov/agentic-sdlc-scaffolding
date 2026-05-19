# Observability and auditability

**Parent:** [OVERVIEW.md](OVERVIEW.md)

---

## LangSmith

- Trace LangGraph runs, tool invocations, and evaluation datasets.
- Tag traces with `case_id`, `workflow_id`, and `step_id` for cross-system correlation.

---

## Logging

- Structured logs (JSON) with **correlation ID** propagated from API → workflow → agent.
- Log levels: ERROR for failures requiring action; INFO for state transitions; DEBUG restricted to
  non-PROD.

---

## Audit tiers

Record events sufficient to reconstruct decisions:

| Tier | What to capture |
|---|---|
| API | Caller identity, route, idempotency key, request/response summary (no secrets) |
| Workflow | State transitions, HITL wait/resume, timeouts |
| HITL | Proposal payload hash or reference, human actor, decision (approve/reject/revise), timestamp |
| LLM | Model id, token usage, latency; store prompt/response per retention policy |

---

## References

- [product-and-hitl.md](product-and-hitl.md)
- [authentication.md](authentication.md)
- [tech-stack.md](tech-stack.md)
