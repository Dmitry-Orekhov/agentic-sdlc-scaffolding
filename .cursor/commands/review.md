# /review — Review Local Changes

Run a comprehensive quality check on all local changes before committing or raising a PR.

## Steps

1. Run `git diff` and `git diff --staged` to enumerate changed files.
2. Run the linter: `<lint-command>`. Report any warnings or errors.
3. Run the type-checker: `<typecheck-command>`. Report any errors.
4. Run the test suite targeting changed files: `<test-command> <changed-files>`.
5. Perform a manual review pass — check each changed file for:
   - [ ] No secrets, credentials, or API keys in diff
   - [ ] No protected paths modified without approval (see `.cursor/rules/04-guardrails.mdc`)
   - [ ] No unrelated files changed (scope discipline)
   - [ ] Error handling is explicit — no empty `catch` blocks
   - [ ] New behavior has corresponding tests
   - [ ] Comments explain *why*, not *what*
6. Summarize findings:
   - **Passed**: list of checks that are clean
   - **Issues found**: list of problems with file + line references
   - **Recommendation**: `ready to commit` | `needs fixes`
