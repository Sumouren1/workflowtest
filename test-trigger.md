# Test Change — Pre-Deploy Review Trigger

This file is used to test the `pre-deploy-review` agentic workflow.

## What this PR tests

- Does the AI Agent auto-trigger on PR open?
- Does the structured review report appear as a PR comment?
- Does the `review: pending` label get applied?

## Simulated change

Added a new utility function (placeholder):

```python
def calculate_discount(price: float, rate: float) -> float:
    """Apply discount rate to price."""
    if rate < 0 or rate > 1:
        raise ValueError("Discount rate must be between 0 and 1")
    return price * (1 - rate)
```

> Merge this PR to verify the review workflow triggers correctly.
