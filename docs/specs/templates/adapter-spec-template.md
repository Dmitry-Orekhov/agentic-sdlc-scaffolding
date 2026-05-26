# Adapter Spec: [Adapter Name]

> Filename convention: `adapters/<adapter-slug>-spec.md` (e.g. `AiGatewayLlmAdapter` → `ai-gateway-llm-adapter-spec.md`)
> One spec per Adapter. An Adapter is the runtime implementation of a Port Protocol.
> Adapters live in `src/adapters/` and are instantiated only by the composition root.
> For examples of how to fill in each section, see [TEMPLATE-README.md](TEMPLATE-README.md).

---

## Purpose

> One paragraph: which external system this Adapter connects to and which Port Protocol it implements.

---

## Port implemented

> The Port Protocol this Adapter satisfies structurally (no subclassing required — P4).

| Port | Location |
|---|---|
| `<PortName>` | `src/ports/<module>.py` |

Port spec: [link to port spec]()

---

## External system

| Property | Value |
|---|---|
| System | `<ExternalSystemName>` |
| SDK / client | `<vendor-sdk>` |
| Adapter location | `src/adapters/<path>.py` |

---

## Configuration

> List the config values the Adapter requires at construction time (credentials, base URLs, timeouts).
> These must not appear on the Port interface (P3).

| Field | Type | Source | Description |
|---|---|---|---|
| `<field_name>` | `<type>` | env / config file | |

---

## Error mapping

> Adapters convert vendor/SDK errors into typed domain errors before surfacing them (P6).
> Adapters never retry — all failure-handling policy belongs in the workflow orchestration layer.

| Vendor exception | Domain error | Condition |
|---|---|---|
| `<VendorException>` | `<DomainError>` | <when> |

---

## Open questions

- [ ] [unresolved question]

---

## References

- [integrations.md](../../architecture/integrations.md)
- [Port spec]()
- [Domain Contract spec]()
