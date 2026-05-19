# Authentication

**Parent:** [OVERVIEW.md](OVERVIEW.md)

---

## Model

**Okta** is the identity provider. Service-to-service access uses **machine-to-machine (M2M)**
OAuth 2.0 client credentials.

| Surface | Requirement |
|---|---|
| Public REST API | Validate `Authorization: Bearer` JWT on every request before handlers |
| Outbound adapters | Obtain client-credentials token scoped for target API |
| Agent tools | Use the same token provider; no ad-hoc API keys in graph code |

---

## Implementation notes

- Centralize token acquisition and refresh in `src/adapters/okta/` (or shared auth module).
- Propagate a **subject** or **client id** into audit context for API calls.
- Align issuer, audience, and JWKS URL with environment config ([config.md](config.md)).

---

## References

- [integrations.md](integrations.md)
- [config.md](config.md)
- [observability.md](observability.md) — audit must not include raw tokens
