# Runtime topology

**Parent:** [OVERVIEW.md](OVERVIEW.md)

---

## Multi-agent execution

- A **workflow orchestrator** plans case/workflow execution and **fans out** work to specialized
  agents.
- The orchestrator **tracks** step status, correlation IDs, and completion.
- Agents should stay **stateless** where possible; durable progress is stored via
  [persistence.md](persistence.md).

---

## Async-first

- Assume **high probability** of parallel steps, I/O-bound adapters, and long-running LLM calls.
- Prefer non-blocking I/O and explicit async boundaries between workflow steps and agent runs.
- Design APIs and internal events so clients can poll or subscribe for completion where needed.

---

## Deployment compatibility

The application must remain **container-portable** and compatible with:

| Target | Use |
|---|---|
| Docker | Local and generic container runs |
| Kubernetes / **OpenShift** | DEV and TEST environments |
| **AWS** (e.g. AgentCore, **ECS**) | PROD |

Orchestration and persistence choices (see ADRs) must not assume a single cloud vendor in
application code—only adapters and configuration differ by environment.

---

## References

- [orchestration.md](orchestration.md)
- [tech-stack.md](tech-stack.md)
- [ADR-001](../adr/ADR-001-workflow-orchestration.md)
- [ADR-002](../adr/ADR-002-openshift-persistence.md)
