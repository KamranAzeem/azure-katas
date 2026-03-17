# Day 1 - Lab 4: Group-Based Access Baseline for Azure Resources

## Context (Why this lab matters)
- SC-300 expects you to connect identity administration to authorization outcomes.

## Objective
Assign Azure RBAC roles to Entra groups and verify effective access.

## Scenario
Support and app teams need different privileges on a shared resource group.

## Direct Portal GUI Steps (Primary)
1. Open Azure Portal -> Resource groups -> rg-sc300.
2. Open Access control (IAM) -> Add role assignment.
3. Assign Reader to sc300-helpdesk.
4. Assign Contributor to sc300-app-users.
5. Open Check access and verify effective access for each test user.

## PowerShell Path (Method #2)
```powershell
Connect-AzAccount
$rg = "rg-sc300"
$helpdesk = Get-AzADGroup -DisplayName "sc300-helpdesk"
$appusers = Get-AzADGroup -DisplayName "sc300-app-users"

New-AzRoleAssignment -ObjectId $helpdesk.Id -RoleDefinitionName Reader -ResourceGroupName $rg
New-AzRoleAssignment -ObjectId $appusers.Id -RoleDefinitionName Contributor -ResourceGroupName $rg

Get-AzRoleAssignment -ResourceGroupName $rg | Format-Table DisplayName,RoleDefinitionName,Scope
```

## Azure CLI Path (Method #3)
```bash
RG=rg-sc300
HELPDESK_OID=$(az ad group show --group sc300-helpdesk --query id -o tsv)
APPUSERS_OID=$(az ad group show --group sc300-app-users --query id -o tsv)

az role assignment create --assignee-object-id "$HELPDESK_OID" --role Reader --resource-group "$RG"
az role assignment create --assignee-object-id "$APPUSERS_OID" --role Contributor --resource-group "$RG"
az role assignment list --resource-group "$RG" --output table
```

## IaC Templates (Optional - ARM)
- ARM reference: labs/arm/day1/scenario01/main.json

## Prerequisites
- rg-sc300 exists in your subscription.
- You have rights to assign RBAC roles at resource group scope.

## Sequence Notes
- Validate group membership first (Lab 1).
- Assign least access first, then elevate only where required.

## Validation
- Reader and Contributor assignments are visible at RG scope.
- Effective access differs correctly between both user paths.

## Complete Cleanup
- Keep baseline assignments for follow-up labs or remove them if required by policy.
