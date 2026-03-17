# Day 2 - Lab 1: Authentication Methods Policy Baseline

## Context (Why this lab matters)
- SC-300 requires day-to-day control of authentication methods and assignment scope.

## Objective
Configure authentication method policy targeting for pilot identities.

## Scenario
Security team wants strong authentication methods enabled for pilot users before broad rollout.

## Direct Portal GUI Steps (Primary)
1. Open Microsoft Entra ID -> Protection -> Authentication methods.
2. Review Microsoft Authenticator, FIDO2, and Temporary Access Pass policy states.
3. Set policy target to pilot group sc300-app-users.
4. Save and note expected sync delay.

## PowerShell Path (Method #2)
```powershell
Connect-MgGraph -Scopes "Policy.ReadWrite.AuthenticationMethod"
Get-MgPolicyAuthenticationMethodPolicy
```

## Azure CLI Path (Method #3)
```bash
az rest --method GET --url https://graph.microsoft.com/beta/policies/authenticationMethodsPolicy
```

## IaC Templates (Optional - ARM)
- Authentication method policy is Graph-managed and not ARM-native.

## Prerequisites
- Pilot group from Day 1 exists.
- Entra role with policy management permissions.

## Sequence Notes
- Use a pilot group first.
- Avoid tenant-wide rollout before log validation.

## Validation
- Policy target reflects pilot group.
- Enabled methods match intended baseline.

## Complete Cleanup
- Keep policy if used as baseline, otherwise revert scope.
