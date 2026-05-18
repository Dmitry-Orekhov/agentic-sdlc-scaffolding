---
description: Perform a thorough AI code review. Use when the user asks to review code, find issues, audit a PR, or check for bugs.
---

# Code Review

Perform a structured review of code changes, focusing on correctness, security, maintainability, and alignment with project conventions.

## Review Scope

Determine the diff to review:
```bash
# Review uncommitted local changes
git diff

# Review a specific PR
gh pr diff <number>

# Review a branch against main
git diff main...<branch>
```

## Review Dimensions

Work through each dimension in order:

### 1. Correctness
- Does the code do what it claims to do?
- Are edge cases (null, empty, boundary values) handled?
- Is error handling explicit — no swallowed exceptions?
- Are async operations awaited correctly?

### 2. Security
- No hardcoded credentials, tokens, or secrets
- Input validation before use in queries or shell commands
- No SQL injection, XSS, or path traversal vectors
- Dependencies are pinned to specific versions

### 3. Test Coverage
- New functionality has corresponding tests
- Bug fixes include regression tests
- Tests cover meaningful behavior, not just happy paths

### 4. Code Style
- Follows conventions in `.cursor/rules/01-code-style.mdc`
- Functions are small and single-purpose
- Names are clear and intention-revealing

### 5. Architecture & Scope
- Change is within the stated task scope
- No unrelated files modified
- Introduces no new circular dependencies

## Output Format

```
## Code Review Summary

### ✅ Passed
- <check>: <brief explanation>

### ⚠️ Suggestions (non-blocking)
- `<file>:<line>`: <issue> → <recommendation>

### ❌ Issues (must fix before merge)
- `<file>:<line>`: <issue> → <recommendation>

### Verdict
`approve` | `request changes` | `needs discussion`
```
