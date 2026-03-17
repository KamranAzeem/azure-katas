# Day 2 - Lab 3: Conditional Access Baseline Policy

## Context (Why this lab matters)
- Conditional Access policy design and safe rollout are core SC-300 responsibilities.

## Objective
Create a report-only MFA policy with safe exclusions and validate impact.

## Scenario
You need to enforce MFA while avoiding accidental tenant lockout.

## Direct Portal GUI Steps (Primary)
1. Open Microsoft Entra ID -> Protection -> Conditional Access -> Policies -> New policy.
2. Name policy ca-sc300-mfa-baseline.
3. Include pilot group, exclude break-glass account.
4. Select cloud apps (all cloud apps or pilot app).
5. Grant control: Require multifactor authentication.
6. Enable as Report-only and save.
7. Review Insights and reporting for hit/miss behavior.

## PowerShell Path (Method #2)
```powershell
Connect-MgGraph -Scopes "Policy.ReadWrite.ConditionalAccess"
Get-MgIdentityConditionalAccessPolicy
```

## Azure CLI Path (Method #3)
```bash
az rest --method GET --url https://graph.microsoft.com/v1.0/identity/conditionalAccess/policies
```

## IaC Templates (Optional - ARM)
- Conditional Access is Graph identity policy, not ARM-native.

## Prerequisites
- Break-glass account identified and excluded.
- Pilot group prepared from Day 1.

## Sequence Notes
- Always start in report-only mode.
- Capture policy rationale and exclusions in notes.

## Quick Run (Safety Check)
```bash
az rest --method GET --url https://graph.microsoft.com/v1.0/identity/conditionalAccess/policies --query "value[?contains(displayName,'sc300')].{Name:displayName,State:state}"
```

## Validation
- Policy exists and state is reportOnly.
- User scope, app scope, and grant controls are correct.
- Break-glass exclusion is present.

## Complete Cleanup
- Keep report-only policy for later labs or disable/delete after practice.
