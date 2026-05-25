# Multi-Agent System — Architecture Overview

> **Audience:** Engineers and AI agents implementing the **runtime multi-agent application**.
>
> **Not covered here:** Cursor/SDLC development workflow — see [`AGENTS.md`](../../AGENTS.md) at the
> repository root. Do not conflate repository dev agents with runtime workflow agents.

This folder documents the **baseline product architecture**. Read this file first, then open the
linked sibling documents for the area you are implementing.

---

## Executive summary

The platform is a **multi-agentic case/workflow system** built on **Augmented Intelligence**: humans
hold tacit knowledge and make final decisions; agents perform supportive automation (intake, parsing,
validation, enrichment, Q&A, elicitation) and submit **proposals** for human **sign-off**.

**Human-in-the-loop (HITL)** is on the **critical path**, not an exception. Execution is **async and
multi-agent**: a workflow orchestrator runs business sub-DAGs; within each step, **LangGraph**
orchestrates agent reasoning. The codebase is a **single monorepo** (Python), deployed to
**OpenShift** (DEV/TEST) and **AWS** (PROD), with external systems reached only through
**ports and adapters**.

---

## Architecture documents

| Topic | Document | Summary |
|---|---|---|
| Product model & HITL | [product-and-hitl.md](product-and-hitl.md) | Augmented Intelligence, human sign-off, elicitation |
| Runtime topology | [runtime-topology.md](runtime-topology.md) | Multi-agent, async-first, deployment targets |
| Orchestration | [orchestration.md](orchestration.md) | Two tiers: workflow sub-DAGs + LangGraph |
| Technology stack | [tech-stack.md](tech-stack.md) | Python, LangChain ecosystem, environments |
| Repository layout | [repo-layout.md](repo-layout.md) | Single repo, module boundaries, planned paths |
| External integrations | [integrations.md](integrations.md) | Okta, AI Gateway, SORs, port–adapter rules |
| Authentication | [authentication.md](authentication.md) | Okta M2M for API and outbound calls |
| Data & public API | [data-and-contracts.md](data-and-contracts.md) | REST, domain contracts, SOR metadata |
| Agent state persistence | [persistence.md](persistence.md) | DynamoDB (PROD), OpenShift store (DEV/TEST) |
| Configuration | [config.md](config.md) | `.env` locally, Vault in TEST/PROD |
| Observability & audit | [observability.md](observability.md) | LangSmith, structured logs, audit tiers |
| Quality & delivery | [quality-and-delivery.md](quality-and-delivery.md) | Mocked ports, unit/integration tests |

For a **new feature or subsystem**, add a dedicated doc from [`TEMPLATE.md`](TEMPLATE.md) and link
it in the table above.

---

## Architecture Decision Records

Immutable decisions live in [`docs/adr/`](../adr/). Supersede via a new ADR; do not rewrite
accepted records.

| ADR | Title | Status |
|---|---|---|
| [ADR-001](../adr/ADR-001-workflow-orchestration.md) | Workflow-level orchestration (sub-DAGs) | accepted |
| [ADR-002](../adr/ADR-002-openshift-persistence.md) | Agent state persistence on OpenShift | accepted |
| [ADR-003](../adr/ADR-003-api-versioning-idempotency.md) | Public REST versioning & idempotency | accepted |
| [ADR-004](../adr/ADR-004-domain-contracts-pydantic.md) | Domain contracts (Pydantic) | accepted |
| [ADR-005](../adr/ADR-005-reasoning-flow-skills.md) | Reasoning-flow Skills as composition unit | accepted |
| [ADR-006](../adr/ADR-006-in-agent-hitl-placement.md) | In-Agent HITL placement | proposed |

---

## Cross-cutting rules (all implementers)

1. **HITL is mandatory** on paths that change external SOR state or send customer-facing outcomes
   unless an ADR documents a specific exception.
2. **Two orchestration tiers** — workflow engine owns case lifecycle; LangGraph owns in-step reasoning.
   Do not merge these responsibilities.
3. **Ports outward only** — new external system → port interface, adapter, fake adapter, tests.
4. **Async by default** — design steps and I/O for concurrent agent work where safe.
5. **No secrets in repo** — use `.env` for local dev and Vault (or platform secrets) for TEST/PROD.
6. **Protected infrastructure paths** — do not change `infra/`, `k8s/`, Dockerfiles, CI, or
   migrations without explicit human approval (repository guardrails).

---

## Open questions

- [ ] Email intake protocol and document retention policy
- [ ] AI Gateway retry, rate-limit, and fallback policy
- [ ] Exact AWS PROD service mapping (AgentCore vs ECS) — operational, not application logic

Track resolution in a new ADR or in the relevant sibling doc once decided.
