# Configuration

**Parent:** [OVERVIEW.md](OVERVIEW.md)

---

## By environment

| Environment | Source | Notes |
|---|---|---|
| Local / DEV | `.env` + typed settings | `.env` is gitignored; never commit secrets |
| TEST / PROD | **Vault** (or platform secret store) | Injected at deploy; no secrets in images |

---

## Configuration categories

- Okta issuer, client id, scopes (references only—values in Vault)
- AI Gateway base URL and timeouts
- Persistence connection (Postgres DSN vs DynamoDB table names)
- Temporal namespace and task queue names
- LangSmith project and API key
- Feature flags for optional adapters

---

## Rules

1. Load settings once at process start via `src/config/`.
2. Do not read `.env` in TEST/PROD paths.
3. Architecture docs and agent output must **never** contain secret values.

---

## References

- [authentication.md](authentication.md)
- [persistence.md](persistence.md)
- [tech-stack.md](tech-stack.md)
