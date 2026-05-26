# Observability Spec: [Skill or Agent Name]

> Filename convention: `observability/<skill-or-agent-slug>-observability-spec.md`
> One spec per Skill or Agent. Documents required trace tags, log events, and audit records.
> See observability.md for platform-wide requirements.
> For examples of how to fill in each section, see [TEMPLATE-README.md](TEMPLATE-README.md).

---

## Purpose

> One paragraph: what observability this component must emit, and what operational or audit
> questions those signals answer.

---

## LangSmith trace tags

> Mandatory on every run before the first LLM call.

| Tag | Value source | Notes |
|---|---|---|
| `case_id` | Parent Agent state | Correlates to Temporal workflow |
| `workflow_id` | Parent Agent state | |
| `step_id` | Parent Agent state | Identifies this Skill invocation |
| [additional tag] | | |

---

## Structured log events

> JSON log events emitted at key state transitions. Use `INFO` for transitions, `ERROR` for
> failures requiring action. `DEBUG` is non-PROD only.
> See [TEMPLATE-README.md](TEMPLATE-README.md) for default Skill log event patterns.

| Event | Level | Fields | Trigger |
|---|---|---|---|
| `<event>` | | | |

---

## Audit records

> Events that must be captured for decision reconstruction. Reference audit tiers from
> observability.md.
> See [TEMPLATE-README.md](TEMPLATE-README.md) for default Skill audit record patterns.

| Tier | Event | What to capture |
|---|---|---|
| `<tier>` | | |

---

## Tool invocation tracing

> LangSmith traces Tool calls automatically. List Tools whose traces need additional metadata.

| Tool | Additional metadata |
|---|---|
| `<ToolName>` | [any extra fields beyond default] |

---

## Open questions

- [ ] [unresolved question]

---

## References

- [observability.md](../../architecture/observability.md)
- [agents.md](../../architecture/agents.md)
- [Skill spec]()
