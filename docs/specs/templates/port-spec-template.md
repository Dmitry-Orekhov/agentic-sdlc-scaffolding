# Port Spec: [Port Name]

> Filename convention: `ports/<port-name-as-slug>-spec.md` (e.g. `LlmPort` → `llm-port-spec.md`)
> `PortName` is derived from `[Port Name]`: PascalCase + `Port` suffix (e.g. `Llm` → `LlmPort`).
> One spec per Port Protocol. Ports live in `src/ports/`.
> See integrations.md for port–adapter design principles (P1–P6).
> For examples of how to fill in each section, see [TEMPLATE-README.md](TEMPLATE-README.md).

---

## Purpose

> One paragraph: what external capability this Port abstracts and which external system it fronts.

---

## Protocol definition

> See [TEMPLATE-README.md — Protocol definition](TEMPLATE-README.md#port-spec-template--protocol-definition) for a code example.

| Method | Input | Output | Raises | Description |
|---|---|---|---|---|
| `<method_name>` | `<InputType>` | `<OutputType>` | `<DomainError>` | |

---

## Design constraints

> Reference the applicable principles from integrations.md.

- **P2** — All types in signatures are domain models from `src/domain/`; no vendor SDKs.
- **P3** — Method signatures express domain intent; no technology-specific parameters.
- **P4** — Satisfied by structural typing (`Protocol`); adapters need not subclass.

---

## Fake

> See [TEMPLATE-README.md — Fake](TEMPLATE-README.md#port-spec-template--fake) for a code example.

| Artifact | Location |
|---|---|
| Fake adapter | `tests/fakes/fake_<port-name-as-slug>.py` |

> Describe the fake's behaviour: what state it holds, what it returns by default,
> and any configurable failure modes.

---

## Error contract

> Domain errors this Port's adapters must raise. Adapters convert vendor errors to these
> before surfacing them — callers never see vendor exceptions (P6).

| Error | Defined in | Condition |
|---|---|---|
| `<DomainError>` | `src/domain/` | [when] |

---

## Open questions

- [ ] [unresolved question]

---

## References

- [integrations.md](../../architecture/integrations.md)
- [Domain Contract spec]()
