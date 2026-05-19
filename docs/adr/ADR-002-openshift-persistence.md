# ADR-002: Agent state persistence (OpenShift vs AWS)

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

The platform persists:

- **Temporal** workflow state (managed by Temporal server, not application-owned)
- **LangGraph checkpoints** and thread metadata for agent resume after HITL
- Application **case pointers** and audit references

**AWS PROD** baseline specifies **DynamoDB**. **OpenShift DEV/TEST** has no managed DynamoDB;
a persistence backend compatible with Kubernetes and LangGraph checkpointing is required.

Application code must not hard-code a single vendor SDK across environments.

---

## Decision drivers

- LangGraph-supported checkpoint backends
- Operational familiarity on OpenShift (Postgres commonly available)
- Parity of **resume-after-HITL** behavior between DEV/TEST and PROD
- Single persistence **abstraction** in code; pluggable backends
- Avoid running DynamoDB-compatible appliances in-cluster unless necessary

---

## Options considered

### Option A: PostgreSQL on OpenShift; DynamoDB on AWS PROD

**Description:** `src/persistence/` exposes one interface. OpenShift uses Postgres (LangGraph
`AsyncPostgresSaver` or equivalent). PROD uses a DynamoDB implementation for checkpoints and app
metadata.

**Pros:**
- Native OpenShift service (StatefulSet or managed Postgres)
- Aligns with common LangGraph deployment guides
- PROD uses purpose-built AWS store

**Cons:**
- Two backend implementations to maintain and test

---

### Option B: PostgreSQL everywhere (including PROD)

**Description:** Single backend; RDS Postgres in AWS PROD.

**Pros:**
- One checkpoint implementation

**Cons:**
- Deviates from stated PROD baseline (DynamoDB)
- RDS operations differ from DynamoDB scaling model already assumed for PROD

---

### Option C: In-cluster DynamoDB Local / custom store

**Description:** Emulate DynamoDB on OpenShift.

**Pros:**
- Identical API in all environments

**Cons:**
- Not production-representative; operational overhead; discouraged for DEV/TEST

---

## Decision

**We choose Option A — PostgreSQL on OpenShift (DEV/TEST) and DynamoDB on AWS (PROD)** because
it matches environment constraints, supports LangGraph checkpointing on OpenShift, and honors the
PROD DynamoDB baseline while keeping vendor specifics inside `src/persistence/`.

---

## Consequences

### Positive

- DEV/TEST teams can use standard Postgres tooling and backups
- PROD retains serverless scaling characteristics of DynamoDB where required
- Clear module boundary for persistence

### Negative / Trade-offs

- Integration tests must run against both fakes and, where needed, backend-specific test containers
- Schema/migration discipline for Postgres; DynamoDB table design and GSIs for PROD

### Neutral

- Temporal retains its own persistence; this ADR covers application and LangGraph checkpoints

---

## Implementation notes

- Define `CheckpointStore` (or equivalent) protocol in `src/persistence/`.
- Implement `PostgresCheckpointStore` and `DynamoDBCheckpointStore`.
- Provide `InMemoryCheckpointStore` for unit tests.
- Connection strings and table names from [config.md](../architecture/config.md) only.
- Never share PROD DynamoDB credentials with DEV clusters.

**Related architecture:** [persistence.md](../architecture/persistence.md)
