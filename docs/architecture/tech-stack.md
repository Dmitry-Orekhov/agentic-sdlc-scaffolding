# Technology stack

**Parent:** [OVERVIEW.md](OVERVIEW.md)

---

## Core

| Layer | Choice |
|---|---|
| Language | Python 3.11+ |
| Agent framework | LangChain, **LangGraph**, **LangSmith** |
| Workflow engine | **Temporal** ([ADR-001](../adr/ADR-001-workflow-orchestration.md)) |
| Domain models | **Pydantic v2** ([ADR-004](../adr/ADR-004-domain-contracts-pydantic.md)) |
| Configuration | Typed settings library (e.g. pydantic-settings) |

---

## Environments

| Environment | Hosting | Notes |
|---|---|---|
| DEV | OpenShift | Full stack; persistence per [ADR-002](../adr/ADR-002-openshift-persistence.md) |
| TEST | OpenShift | Secrets from Vault |
| PROD | AWS | Agent state in **DynamoDB**; containers on AgentCore/ECS |

---

## Identity and AI services

- **Okta** — M2M tokens ([authentication.md](authentication.md))
- **AI Gateway** — single API surface for LLM and KB/RAG ([integrations.md](integrations.md))

---

## References

- [runtime-topology.md](runtime-topology.md)
- [config.md](config.md)
- [observability.md](observability.md)
