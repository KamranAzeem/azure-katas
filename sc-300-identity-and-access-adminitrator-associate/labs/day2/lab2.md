# Day 2 - Lab 2: SSPR and Combined Registration

## Context (Why this lab matters)
- SSPR and registration controls are recurring SC-300 implementation and troubleshooting tasks.

## Objective
Enable SSPR for a selected scope and validate registration controls.

## Scenario
Helpdesk wants to reduce password reset tickets for pilot users.

## Direct Portal GUI Steps (Primary)
1. Open Microsoft Entra ID -> Password reset.
2. Set self-service password reset to Selected.
3. Select pilot group (sc300-helpdesk or sc300-app-users).
4. Review Authentication methods and required options.
5. Open Registration and confirm combined registration experience is enabled.

## PowerShell Path (Method #2)
```powershell
Connect-MgGraph -Scopes "Policy.Read.All","Policy.ReadWrite.Authorization"
Get-MgPolicyAuthorizationPolicy
```

## Azure CLI Path (Method #3)
```bash
az rest --method GET --url https://graph.microsoft.com/beta/policies/authorizationPolicy
```

## IaC Templates (Optional - ARM)
- SSPR policy is tenant identity configuration and not ARM-native.

## Prerequisites
- At least one pilot group with active users.

## Sequence Notes
- Keep rollout scoped until tested.
- Document original tenant setting before changing it.

## Validation
- SSPR scope is selected group, not broad all-users.
- Registration settings are visible and aligned to policy.

## Complete Cleanup
- Revert to original SSPR scope if this was a temporary exercise.
