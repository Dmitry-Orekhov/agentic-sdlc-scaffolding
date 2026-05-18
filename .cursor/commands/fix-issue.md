# /fix-issue [number] — Fix a GitHub Issue

Fetch an issue, implement a fix, and open a pull request.

## Steps

1. Fetch issue details:
   ```bash
   gh issue view <number> --json title,body,labels,assignees
   ```
2. Understand the reported problem — read the issue body carefully.
3. Use Plan Mode (`Shift+Tab`) to research the codebase and create an implementation plan.
   Save the plan to `.cursor/plans/YYYY-MM-DD-fix-<number>.md`.
4. Create a fix branch:
   ```bash
   git checkout -b fix/<slug>-<number>
   ```
5. Implement the fix, following patterns in `.cursor/rules/`.
6. Write a regression test that covers the exact failure path from the issue.
7. Run the full verification loop:
   ```bash
   <typecheck-command>
   <test-command>
   <lint-command>
   ```
8. Commit:
   ```
   fix(<scope>): <short description>

   Closes #<number>
   ```
9. Run `/pr` to push and open the pull request.

## Notes

- Reference the issue number in both the commit footer (`Closes #<n>`) and the PR body.
- If the root cause differs from what the issue describes, note this in the PR body.
