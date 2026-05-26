# Tool Spec: [Tool Name]

> Filename convention: `tools/<tool-slug>-spec.md`
> One spec per LangChain Tool. Tools live in `src/agents/tools/` and are shared across Skills.
> See integrations.md P1: agents call external services via Tools only.
> For examples of how to fill in each section, see [TEMPLATE-README.md](TEMPLATE-README.md).

---

## Purpose

> One paragraph: what capability this Tool exposes to the LLM, which Skills use it,
> and which Port it wraps.

---

## Interface

### Name and description

> The `name` and `description` are exposed to the LLM for tool selection — be precise.

- **name:** `<tool_name>`
- **description:** `<what the LLM should understand about when and how to use this tool>`

### Input schema

```python
class <ToolName>Input(BaseModel):
    field_name: FieldType = Field(..., description="...")
```

### Output

> Domain type returned. Must be a model from `src/domain/`.

```python
<DomainType>
```

---

## Port dependency

> The Tool declares a dependency on a Port (Protocol) as its type. At runtime the composition
> root injects an Adapter; in tests it injects a Fake. The Tool is unaware of which backing is active.
> See [TEMPLATE-README.md — Port dependency](TEMPLATE-README.md#tool-spec-template--port-dependency) for examples.

| Port | Method called | Location |
|---|---|---|
| `<PortName>` | `<method_name>` | `src/ports/<module>.py` |

| Context | Injected implementation |
|---|---|
| Runtime | `src/adapters/<adapter_module>.py` |
| Unit / CI tests | `tests/fakes/fake_<port_slug>.py` |

---

## LangSmith observability

> Tool invocations are traced automatically. List any additional tags or metadata to attach.
> See [TEMPLATE-README.md — Implementation sketch](TEMPLATE-README.md#tool-spec-template--implementation-sketch) for a code pattern.

- Automatic: tool name, inputs, output, latency
- Additional: [list any]

---

## Open questions

- [ ] [unresolved question]

---

## References

- [agents.md](../../architecture/agents.md)
- [integrations.md](../../architecture/integrations.md)
- [Port spec]()
- [Skill spec(s)]()
