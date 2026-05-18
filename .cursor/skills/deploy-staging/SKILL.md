---
description: Deploy to staging environment. Use when the user asks to deploy, ship, push to staging, or release a staging build.
---

# Deploy to Staging

## Prerequisites

- All tests pass locally
- PR is approved and merged to `develop`
- You have credentials for the deployment target (injected via environment — never hardcoded)

## Steps

1. Confirm the working branch and latest changes:
   ```bash
   git status
   git log --oneline -5
   ```
2. Run the full pre-deploy verification:
   ```bash
   <typecheck-command>
   <test-command>
   <lint-command>
   <build-command>
   ```
3. Execute the staging deploy:
   ```bash
   <deploy-staging-command>
   ```
4. Wait for the deployment to complete and verify health:
   ```bash
   curl -sf https://staging.<your-domain>/health | jq .
   ```
5. Run smoke tests against the staging environment:
   ```bash
   <smoke-test-command> --env staging
   ```
6. Report status:
   - **Success**: deployment URL, health check response, smoke test results
   - **Failure**: error output, rollback steps taken

## Rollback

If the deployment fails or health checks do not pass:
```bash
<rollback-command>
```
Notify the team in the project Slack channel: `#deployments`.

## Notes

- Do **not** deploy to production using this skill — production deployments require a separate approval workflow.
- Never deploy directly from a feature branch; always deploy from `develop` or a release branch.
