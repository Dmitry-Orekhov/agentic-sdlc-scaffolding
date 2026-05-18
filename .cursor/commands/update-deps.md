# /update-deps — Update Dependencies

Safely update outdated dependencies one at a time, verifying correctness after each update.

## Steps

1. Identify outdated packages:
   ```bash
   # npm / pnpm / yarn
   npm outdated
   # pip
   pip list --outdated
   # cargo
   cargo outdated
   ```
2. For each outdated package (update **one at a time**):
   a. Review the package changelog / release notes for breaking changes.
   b. Update the package:
      ```bash
      npm install <package>@latest
      ```
   c. Run the full verification loop:
      ```bash
      <typecheck-command>
      <test-command>
      <lint-command>
      ```
   d. If all checks pass, continue to the next package.
   e. If a check fails, investigate and fix before proceeding.
3. When all updates are done, commit:
   ```
   chore(deps): update <list of packages> to latest
   ```
4. Run `/pr` to open a pull request.

## Notes

- **Never** batch-update all packages at once — it makes root-cause analysis impossible.
- Major version bumps require extra scrutiny; note any breaking changes in the PR body.
- Do not update packages in `infra/` or lock files for deployment tooling without explicit approval.
