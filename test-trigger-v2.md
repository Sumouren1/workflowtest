# Test v2 — Pre-Deploy Review Trigger

This PR tests the **compiled** pre-deploy-review workflow (v2).

## What changed from v1

The previous attempt failed because the `.md` workflow file was not compiled
into a `.lock.yml` file. GitHub Agentic Workflows require two files:

1. `pre-deploy-review.md` — natural language workflow definition
2. `pre-deploy-review.lock.yml` — compiled GitHub Actions YAML (the actual trigger)

This PR verifies the fix is working.

## Expected behavior after merge

When this PR is **opened**, the AI Agent should:
1. Auto-trigger via the `pull_request` event in `pre-deploy-review.lock.yml`
2. Analyze the changed files
3. Post a structured review comment with risk level and checklist
4. Apply a `review-pending` or `review-blocked` label
