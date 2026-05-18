---
description: Onboard to this codebase. Use when the user asks how the project works, wants a codebase tour, or needs to understand the architecture before starting work.
---

# Codebase Onboarding

Provide a structured orientation to the repository for a new developer or agent.

## Steps

1. Read `AGENTS.md` for the project overview, commands, and conventions.
2. Read `.cursor/rules/` to understand always-on coding standards.
3. Map the top-level directory structure:
   ```bash
   ls -la
   ```
4. Identify the entry point(s) of the application:
   - For a web server: look for `main.*`, `index.*`, `server.*`, or `app.*`
   - For a library: look for `src/index.*` or `lib/index.*`
5. Trace one end-to-end flow (e.g. an HTTP request from router → controller → service → DB):
   - Use semantic search or grep to find route definitions
   - Follow the call chain through the codebase
6. Review the test structure to understand coverage patterns.
7. Check `docs/architecture/` for system diagrams and `docs/adr/` for key decisions.

## Output

Produce a concise onboarding summary:

```
## Project Overview
<1–2 sentences>

## Tech Stack
- Runtime: ...
- Framework: ...
- Database: ...
- Test runner: ...

## Key Entry Points
- `<path>`: <purpose>

## Core Flow: <flow-name>
<file> → <file> → <file>

## Conventions to Know
- <convention 1>
- <convention 2>

## Where to Start
<recommended first file or task for a new contributor>
```
