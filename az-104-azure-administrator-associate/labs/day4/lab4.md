# Day 4 - Lab 4: Governance Controls (Locks, Cost Signals, Advisor)

## Context (Why this lab matters)
- AZ-104 governance coverage extends beyond tags/policy to locks, cost visibility, and recommendations.
- This lab adds practical governance controls often seen in exam scenarios.
- You should finish with clear details of lock behavior and cost-governance signal review.

## Objective
Apply and validate governance controls using resource locks, budget/consumption visibility, and Azure Advisor recommendations.

## Scenario
Your team wants guardrails to prevent accidental changes and improve cost governance.
You must apply a lock, inspect cost controls, and summarize Advisor recommendations.

## Direct Portal GUI Steps (Primary)
1. Open Azure Portal -> **Resource groups** -> **rg-az104**.
2. Open **Locks** -> **Add** and create a lock:
  - Lock name: `lock-d4-readonly-test`
   - Lock type: `Read-only` (or `CanNotDelete` if preferred)
3. Attempt one safe change in the RG to observe lock behavior, then note down the result.
4. Open **Subscriptions** -> your subscription, then review both areas separately:
  - **Cost Management** (cost analysis, budgets, advisor)
  - **Billing** (invoice/profile views, if available in your subscription type)
5. Open **Budgets** and review existing budgets; create a small test budget if permitted.
6. Open **Advisor** and review at least one recommendation category (cost/reliability/security/operational excellence/performance).
7. Open **Resource groups** -> **rg-az104** and confirm governance details (lock state + related activity log entries).
8. Note down details: lock type/scope, cost visibility outcome, and Advisor recommendations reviewed.

## Azure CLI Helper
```bash
RG=rg-az104
az account show --output table
az group show --name $RG --output table
az lock list --resource-group $RG --output table
az advisor recommendation list --query "[].{Category:category,Impact:impact,ShortDescription:shortDescription.solution}" --output table
```

## Optional Deep-Dive Exercise (Lock Enforcement Proof)
Use this only if you want explicit guardrail behavior details.

### Create
1. Add lock `lock-d4-readonly-test` at RG scope.
2. Choose one low-risk control-plane action (for example update RG tag).

### Validate
1. Perform the action while lock is active and capture outcome/error.
2. Remove lock and retry the same action.
3. Confirm action succeeds after lock removal.

### Cleanup
1. Remove temporary lock used only for this test.
2. Remove temporary budget (if created only for this lab and not needed).

## IaC Templates (Optional - Later)
- Day 4 governance labs are Portal-first by design.
- Optional ARM scenario templates can be added later if needed.

## Prerequisites
- `rg-az104` in `northeurope`
- Permission to manage locks and view cost/advisor data in the subscription
- If RG is missing: `az group create --name rg-az104 --location northeurope --output table`

## Sequence Notes
- Primary path: complete Portal steps first; use CLI helper for verification.
- Cost/Budget features can vary by subscription type; if unavailable, note down the blocker and continue.
- Avoid destructive tests while lock is active unless the step explicitly calls for it.

## Quick Run (Azure CLI)
```bash
RG=rg-az104
LOCK=lock-d4-readonly-test
az account show --output table
az group show --name $RG --output table
az lock create --name $LOCK --lock-type ReadOnly --resource-group $RG --notes "AZ-104 Day 4 lock behavior test"
az lock list --resource-group $RG --output table
az advisor recommendation list --output table
```

### One-Liner (copy/paste)
```bash
RG=rg-az104 && LOCK=lock-d4-readonly-test && az account show --output table && az group create --name $RG --location northeurope --output table && az group show --name $RG --output table && az lock create --name $LOCK --lock-type ReadOnly --resource-group $RG --notes "AZ-104 Day 4 lock behavior test" && az lock list --resource-group $RG --output table && az advisor recommendation list --output table
```

## Exercise
1. Create RG lock `lock-d4-readonly-test`.
2. Validate lock presence via Portal and CLI.
3. Review subscription budgets/cost view and Advisor recommendations.
4. Capture a short governance summary (lock + cost + advisor outcomes).

## Validation
- Lock is visible at RG scope and behavior is observed/documented.
- At least one cost-governance signal is reviewed (budget or equivalent cost visibility).
- At least one Advisor recommendation category is reviewed and noted.

## Complete Cleanup
- Remove temporary lock:
  - `az lock delete --name lock-d4-readonly-test --resource-group rg-az104`
- Remove temporary budget only if it was created solely for this lab.
