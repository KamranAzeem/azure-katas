# Day 4 - Lab 1: PIM Role Eligibility Baseline

## Context (Why this lab matters)
- SC-300 governance objectives include just-in-time privileged access using PIM.

## Objective
Configure role eligibility and activation controls in PIM.

## Scenario
Permanent privileged access must be replaced with eligible assignment and controlled activation.

## Direct Portal GUI Steps (Primary)
1. Open Microsoft Entra ID -> Privileged Identity Management.
2. Select Microsoft Entra roles.
3. Choose User Administrator (or equivalent test role).
4. Add eligible assignment for test admin user.
5. Configure activation requirements (MFA, justification, max duration).

## PowerShell Path (Method #2)
```powershell
Connect-MgGraph -Scopes "RoleManagement.ReadWrite.Directory"
Get-MgRoleManagementDirectoryRoleEligibilitySchedule
```

## Azure CLI Path (Method #3)
```bash
az rest --method GET --url https://graph.microsoft.com/v1.0/roleManagement/directory/roleEligibilitySchedules
```

## IaC Templates (Optional - ARM)
- PIM schedules and policies are Graph-governance resources.

## Prerequisites
- Entra ID P2 licensing for full PIM feature set.

## Sequence Notes
- Use test accounts only.
- Keep at least one emergency access account outside restrictive workflows.

## Validation
- Eligible assignment is visible for selected role.
- Activation rules are enforced.

## Complete Cleanup
- Remove temporary eligible assignments created for practice.
