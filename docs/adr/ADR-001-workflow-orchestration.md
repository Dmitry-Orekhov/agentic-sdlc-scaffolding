# ADR-001: Workflow-level orchestration (sub-DAGs)

---

## Metadata

| Field | Value |
|---|---|
| Status | `accepted` |
| Date | 2026-05-18 |
| Deciders | Architecture baseline |
| Supersedes | — |
| Superseded by | — |

---

## Context

The multi-agent platform requires **durable, long-running case workflows** composed of sub-DAGs
with explicit **human-in-the-loop** waits (elicitation, review, sign-off). Agent **reasoning**
within a single step is handled separately by **LangGraph** (Tier 2).

Tier 1 must support: async activities, timers, compensation, visibility into running cases, and
signals from humans or external systems—without embedding LLM logic in the workflow engine.

The system deploys to **OpenShift** (DEV/TEST) and **AWS** (PROD) using **Python** services in
containers.

---

## Decision drivers

- Durable execution across pod restarts and deploys
- First-class **signals** for HITL resume (human input, approval, rejection)
- Python SDK maturity and team alignment with LangChain ecosystem
- Operable on Kubernetes/OpenShift and AWS without vendor lock-in at the code level
- Clear separation from LangGraph (Tier 2) responsibilities

---

## Options considered

### Option A: Temporal

**Description:** Durable workflow engine with Python SDK; workflows as code; activities invoke
LangGraph and adapters.

**Pros:**
- Built for long-running processes, signals, and timers
- Strong operational tooling and visibility
- Widely used for HITL and saga patterns
- Runs on Kubernetes; Temporal Cloud optional on AWS

**Cons:**
- Additional infrastructure component (Temporal cluster)
- Learning curve for workflow determinism rules

---

### Option B: Prefect / Dagster

**Description:** Python-native DAG orchestrators.

**Pros:**
- Familiar DAG model for data teams
- Good scheduling and observability

**Cons:**
- HITL and long-lived human wait states are less natural than Temporal signals
- Geared toward batch/ETL more than interactive case management

---

### Option C: AWS Step Functions (PROD only) + custom OpenShift orchestrator

**Description:** Managed workflows on AWS; separate implementation on OpenShift.

**Pros:**
- Managed service in AWS PROD

**Cons:**
- **Two workflow implementations** violate single-repo/single-behavior goal
- Harder to test consistently across environments

---

### Option D: LangGraph only (single tier)

**Description:** Use LangGraph parent graphs for entire case lifecycle.

**Pros:**
- Fewer moving parts

**Cons:**
- Blurs case lifecycle with LLM reasoning graphs
- Weaker SLA/timer/compensation primitives for enterprise workflows
- Rejected by two-tier architecture baseline

---

## Decision

**We choose Option A — Temporal** because it provides durable workflow execution, first-class HITL
signals, and a single Python codebase for workflow logic across OpenShift and AWS, while keeping
LangGraph focused on agent reasoning inside activities.

---

## Consequences

### Positive

- One workflow model for all environments
- Explicit `workflow.wait_condition` / signal patterns for human gates
- Activities can invoke LangGraph graphs with clear boundaries

### Negative / Trade-offs

- Operate a Temporal cluster (self-hosted on OpenShift; Temporal Cloud or self-hosted on AWS)
- Workflow code must follow determinism constraints (side effects only in activities)

### Neutral

- LangGraph remains Tier 2; Temporal does not replace agent orchestration

---

## Implementation notes

- Place workflow definitions and activities in `src/workflow/`.
- Each activity that runs agents should accept domain types and return proposals, not raw LLM text
  as the sole contract.
- Use distinct task queues per environment (`dev`, `test`, `prod`).
- Document local dev setup (Temporal dev server or docker-compose) in project README when `src/`
  is scaffolded.

**Related architecture:** [orchestration.md](../architecture/orchestration.md)
