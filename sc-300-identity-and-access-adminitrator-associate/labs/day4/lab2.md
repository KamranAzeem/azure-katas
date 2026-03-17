# Day 4 - Lab 2: Access Reviews Baseline

## Context (Why this lab matters)
- Access reviews support regular review and least-access checks.

## Objective
Create and verify access review for security group membership.

## Scenario
A high-impact access group needs quarterly review and clean-up.

## Direct Portal GUI Steps (Primary)
1. Open Microsoft Entra ID -> Identity Governance -> Access reviews -> New access review.
2. Scope review to group sc300-app-users.
3. Select reviewers (owners or named reviewers).
4. Configure recurrence and auto-apply behavior.
5. Start review and verify active review instance.

## PowerShell Path (Method #2)
```powershell
Connect-MgGraph -Scopes "AccessReview.ReadWrite.All"
Get-MgIdentityGovernanceAccessReviewDefinition
```

## Azure CLI Path (Method #3)
```bash
az rest --method GET --url https://graph.microsoft.com/v1.0/identityGovernance/accessReviews/definitions
```

## IaC Templates (Optional - ARM)
- Access review definitions are Graph identity governance items.

## Prerequisites
- Entra ID P2 licensing.
- Review target group exists.

## Sequence Notes
- Keep reviewer assignment explicit.
- Document decision behavior for no-response cases.

## Validation
- Review definition exists and is active.
- Scope, reviewers, and schedule match expected settings.

## Complete Cleanup
- Delete temporary access reviews if not required for ongoing practice.
