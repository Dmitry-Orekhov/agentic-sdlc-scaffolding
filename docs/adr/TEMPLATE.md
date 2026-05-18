# ADR-000: [Short Title of Decision]

> Architecture Decision Record (ADR) — captures an important architectural choice,
> its context, the options considered, and the rationale for the decision taken.
>
> Filename convention: `ADR-<NNN>-<slug>.md`
> Status values: `proposed` | `accepted` | `deprecated` | `superseded by ADR-NNN`

---

## Metadata

| Field | Value |
|---|---|
| Status | `proposed` |
| Date | YYYY-MM-DD |
| Deciders | @author, @reviewer |
| Supersedes | — |
| Superseded by | — |

---

## Context

> Describe the problem, the forces at play, and the situation that makes this decision necessary.
> Include constraints (technical, team, time) that influenced the option space.

---

## Decision Drivers

- <driver 1: e.g., must support horizontal scaling>
- <driver 2: e.g., team has existing expertise in X>
- <driver 3: e.g., must not introduce a new runtime dependency>

---

## Options Considered

### Option A: [Name]

**Description**: ...

**Pros**:
- ...

**Cons**:
- ...

---

### Option B: [Name]

**Description**: ...

**Pros**:
- ...

**Cons**:
- ...

---

### Option C: [Name] *(if applicable)*

**Description**: ...

---

## Decision

> State the chosen option and the key reason it was selected over the alternatives.

**We choose Option [A/B/C]** because <primary rationale>.

---

## Consequences

### Positive
- <benefit 1>
- <benefit 2>

### Negative / Trade-offs
- <trade-off 1>
- <trade-off 2>

### Neutral
- <neutral outcome>

---

## Implementation Notes

> Optional: pointers to relevant code, migration steps, or follow-up tasks.

- Related PR: #<number>
- Follow-up: <task description> tracked in #<issue>
