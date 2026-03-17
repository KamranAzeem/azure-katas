# Day 3 - Lab 3: Managed Identity and Key Vault Access Pattern

## Context (Why this lab matters)
- SC-300 includes workload identities and credentialless secret access design.

## Objective
Enable managed identity and grant least-privilege Key Vault secret access.

## Scenario
A workload must read a secret without storing credentials in code.

## Direct Portal GUI Steps (Primary)
1. Open Azure Portal and create rg-sc300 if missing.
2. Create Key Vault named kv-sc300-<suffix>.
3. Create Automation Account aa-sc300 and enable system-assigned identity.
4. In Key Vault, grant Key Vault Secrets User role to automation managed identity.
5. Create secret sc300-demo-secret.

## PowerShell Path (Method #2)
```powershell
Connect-AzAccount
$rg="rg-sc300"; $loc="northeurope"
New-AzResourceGroup -Name $rg -Location $loc -Force
$kv = New-AzKeyVault -Name "kvsc300$((Get-Random -Maximum 9999))" -ResourceGroupName $rg -Location $loc
$aa = New-AzAutomationAccount -Name "aa-sc300" -ResourceGroupName $rg -Location $loc -AssignSystemIdentity
```

## Azure CLI Path (Method #3)
```bash
RG=rg-sc300
LOC=northeurope
az group create --name $RG --location $LOC --output table
KV=kvsc300$RANDOM
az keyvault create --name $KV --resource-group $RG --location $LOC
az automation account create --name aa-sc300 --resource-group $RG --location $LOC --assign-identity
PRINCIPAL_ID=$(az automation account show --name aa-sc300 --resource-group $RG --query identity.principalId -o tsv)
KV_SCOPE=$(az keyvault show --name $KV --query id -o tsv)
az role assignment create --assignee-object-id "$PRINCIPAL_ID" --role "Key Vault Secrets User" --scope "$KV_SCOPE"
az keyvault secret set --vault-name $KV --name sc300-demo-secret --value "sc300-value"
```

## IaC Templates (Optional - ARM)
- ARM reference: labs/arm/day3/scenario01/main.json

## Prerequisites
- Subscription-level rights to create resource group/resources.

## Sequence Notes
- Ensure identity principal exists before creating role assignment.
- Role propagation may take a few minutes.

## Validation
- Managed identity principalId is populated.
- Role assignment exists at Key Vault scope.
- Secret object exists and is readable by authorized identity.

## Complete Cleanup
- Delete Automation Account and Key Vault when done.
- Keep rg-sc300 only if needed for upcoming labs.
