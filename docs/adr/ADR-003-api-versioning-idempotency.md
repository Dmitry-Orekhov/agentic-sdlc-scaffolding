# ADR-003: Public REST API versioning and idempotency

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

The platform exposes a **public REST API** for case/workflow interaction. Clients include internal
portals and possibly partner systems. Operations are often **retried** (network timeouts, client
libraries) and workflows are **long-running**—duplicate submits must not create duplicate side
effects.

---

## Decision drivers

- Predictable client integration
- Safe retries on mutating operations
- Clear deprecation path for breaking changes
- Minimal magic headers for operators and support teams

---

## Options considered

### Option A: URL path versioning + `Idempotency-Key` header

**Description:** `/v1/...` prefix; clients send `Idempotency-Key: <uuid>` on POST, PUT, PATCH.

**Pros:**
- Visible in logs and support tooling
- Industry-standard idempotency pattern (Stripe-style)
- Cache idempotency responses by key + route + principal

**Cons:**
- URL changes when versioning increments

---

### Option B: `Accept-Version` header only

**Description:** Unversioned paths; version negotiated by header.

**Pros:**
- Stable URLs

**Cons:**
- Harder to route at gateway; easy to misconfigure clients
- Less obvious in access logs

---

### Option C: Idempotency in request body only

**Description:** `client_request_id` field in JSON body.

**Pros:**
- Visible in payload audits

**Cons:**
- Inconsistent across endpoints; easy to omit; harder for generic API gateways

---

## Decision

**We choose Option A — URL path versioning (`/v1/`) and the `Idempotency-Key` header** because
versioning is explicit in routes and logs, and header-based idempotency applies uniformly to all
mutating endpoints without polluting domain models with transport concerns.

### Versioning rules

- Current stable version: **`v1`** — all public routes under `/v1/`.
- Breaking changes require **`v2`** (new prefix); `v1` maintained for a documented deprecation window.
- Non-breaking additions (optional fields, new endpoints) stay in `v1`.

### Idempotency rules

- Required on **POST**, **PUT**, and **PATCH** that create or mutate resources.
- Value: **UUID v4** (or ULID); max length 128 characters.
- Server stores `(principal, method, path, idempotency_key) → response` for **24 hours**.
- Replays with the same key and body return the **original status and body**; mismatched body
  returns **409 Conflict**.
- Optional on GET and DELETE (DELETE may use idempotency where soft-delete creates resources).

---

## Consequences

### Positive

- Clients can safely retry without duplicate cases or duplicate SOR writes triggered via API
- Support can identify API version from URL alone

### Negative / Trade-offs

- Requires idempotency store (Redis, DynamoDB, or Postgres table—implementation detail in `src/api/`)
- Handlers must be written idempotent beyond the header (workflow start uses deterministic workflow id)

### Neutral

- Internal agent-to-service calls use domain contracts, not public idempotency headers

---

## Implementation notes

- Implement middleware in `src/api/` to enforce and record idempotency keys.
- Document required headers in OpenAPI spec when API is scaffolded.
- Align workflow `workflow_id` derivation with idempotency key where API starts workflows.

**Related architecture:** [data-and-contracts.md](../architecture/data-and-contracts.md)
