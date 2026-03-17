# Day 5 - Lab 2: Troubleshooting Sprint

## Context (Why this lab matters)
- SC-300 evaluates real troubleshooting of sign-in failures, policy conflicts, and app access errors.

## Objective
Identify root cause and apply one verified remediation.

## Scenario
Users report MFA loops, denied sign-ins, and app consent issues.

## Direct Portal GUI Steps (Primary)
1. Open Microsoft Entra ID -> Monitoring and health -> Sign-in logs.
2. Select a failed event and inspect Conditional Access details.
3. Review app API permissions and consent status for impacted app.
4. Apply one safe fix and retest sign-in.
5. Capture before/after evidence.

## PowerShell Path (Method #2)
```powershell
Connect-MgGraph -Scopes "AuditLog.Read.All","Policy.Read.All"
Get-MgAuditLogSignIn -Top 20
```

## Azure CLI Path (Method #3)
```bash
az rest --method GET --url "https://graph.microsoft.com/v1.0/auditLogs/signIns?$top=20"
```

## IaC Templates (Optional - ARM)
- Troubleshooting identity policy behavior is not ARM-native.

## Prerequisites
- At least one policy/app/test user path available.

## Sequence Notes
- Change only one control at a time.
- Keep rollback plan for every remediation.

## Validation
- Root cause is documented.
- Remediation is verified with new test result.

## Complete Cleanup
- Revert intentionally introduced misconfigurations.
