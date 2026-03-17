# Day 4 - Lab 3: Entitlement Management Access Package Baseline

## Context (Why this lab matters)
- SC-300 includes access package lifecycle design for internal and external users.

## Objective
Create catalog, access package, and assignment policy.

## Scenario
Users require controlled, time-bound access through approval-based requests.

## Direct Portal GUI Steps (Primary)
1. Open Microsoft Entra ID -> Identity Governance -> Entitlement management.
2. Create catalog cat-sc300-baseline.
3. Add resource (group or app) to catalog.
4. Create access package pkg-sc300-pilot.
5. Configure request policy with approver and expiration window.

## PowerShell Path (Method #2)
```powershell
Connect-MgGraph -Scopes "EntitlementManagement.ReadWrite.All"
Get-MgEntitlementManagementCatalog
```

## Azure CLI Path (Method #3)
```bash
az rest --method GET --url https://graph.microsoft.com/v1.0/identityGovernance/entitlementManagement/catalogs
```

## IaC Templates (Optional - ARM)
- Entitlement items are managed via Graph APIs.

## Prerequisites
- Entra ID P2 licensing.
- Target resources already created.

## Sequence Notes
- Build catalog first.
- Add resources before creating package policy.

## Validation
- Catalog and access package exist.
- Request policy has approval and expiration controls.

## Complete Cleanup
- Remove package/policy if created only for this exercise.
