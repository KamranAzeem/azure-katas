# Day 2 - Lab 4: Password Protection and Smart Lockout

## Context (Why this lab matters)
- Password resilience controls map directly to SC-300 authentication governance objectives.

## Objective
Review password protection posture and lockout controls.

## Scenario
Security requires confirmation that weak password and spray attack protections are configured.

## Direct Portal GUI Steps (Primary)
1. Open Microsoft Entra ID -> Protection -> Authentication methods -> Password protection.
2. Review smart lockout threshold and lockout duration.
3. Review banned password list behavior.
4. Record effective tenant settings.

## PowerShell Path (Method #2)
```powershell
Connect-MgGraph -Scopes "Policy.Read.All"
Get-MgPolicyAuthenticationMethodPolicy
```

## Azure CLI Path (Method #3)
```bash
az rest --method GET --url https://graph.microsoft.com/beta/policies/authenticationMethodsPolicy
```

## IaC Templates (Optional - ARM)
- Password protection controls are Graph-managed policy settings.

## Prerequisites
- Read access to authentication method policies.

## Sequence Notes
- This lab is primarily a verification and documentation exercise.
- Do not change global values unless you have tenant approval.

## Validation
- Current smart lockout and password protection settings are captured.
- Settings align to your policy target.

## Complete Cleanup
- No cleanup needed when performing read-only verification.
