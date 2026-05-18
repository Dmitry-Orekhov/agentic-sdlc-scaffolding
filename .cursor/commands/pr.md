# /pr — Create a Pull Request

Commit staged changes, push, and open a pull request.

## Steps

1. Run `git diff --staged` and `git diff` to review all staged and unstaged changes.
2. Write a commit message following Conventional Commits format:
   `<type>(<scope>): <short imperative summary>`
3. Stage all relevant files with `git add <files>` (do not stage unrelated files).
4. Commit: `git commit -m "<message>"`.
5. Push to the remote branch: `git push -u origin HEAD`.
6. Open a pull request with:
   ```bash
   gh pr create \
     --title "<Conventional Commits title>" \
     --body "$(cat <<'EOF'
   ## What changed
   <summary>

   ## Why
   <motivation>

   ## How to test
   - [ ] <test step 1>
   - [ ] <test step 2>
   EOF
   )"
   ```
7. Return the PR URL to the user.

## Notes

- Target branch: `develop` (not `main`) unless this is a hotfix.
- Do **not** self-merge. The PR requires human review.
- If CI fails after push, investigate with `gh run list` and fix before requesting review.
