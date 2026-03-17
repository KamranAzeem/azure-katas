# Day 1 - Lab 2: Storage Baseline

## Context (Why this lab matters)
- Storage account baseline configuration is a core AZ-104 operational skill.
- This lab focuses on secure defaults and basic data-plane setup you will reuse later.
- You should finish with one validated storage account and container baseline.

## Objective
Deploy a secure baseline storage account in `rg-az104`.

## Scenario
Your team needs a new baseline storage account for upcoming labs.
You create the account with secure defaults and verify container/data-protection readiness.

## Direct Portal GUI Steps (Primary)
1. Open Azure Portal -> **Storage accounts** -> **Create**.
2. Select `rg-az104`, region `North Europe`, and a globally unique storage account name.
3. Set Performance to Standard, and set Redundancy to LRS, and open the account after deployment.
4. Open **Data storage** -> **Containers** and create `labdata`.
5. In the main storage account open **Data protection** (under Data management) and validate blob soft delete/versioning settings.

## Azure CLI Helper
```bash
RG=rg-az104
az account show --output table
az group show --name $RG --output table
az storage account list --resource-group $RG --query "[].{Name:name,Location:location,Sku:sku.name}" --output table
# Replace <storage-account-name> with the account created in Portal
az storage container list --account-name <storage-account-name> --auth-mode login --output table
```

## Optional Deep-Dive Exercise (Storage Protection Proof)
Use this only if you want observable data-protection behavior.

### Create
1. Upload one small test blob into container `labdata`.
2. Delete the blob in Portal or CLI.

### Validate
1. Open storage account -> **Data protection** and confirm soft delete/versioning remain enabled.
2. List deleted blobs or blob versions and confirm the deleted item is recoverable during retention. (lab data -> Drop down column "Show active blobs" -> "Show active and deleted blobs" -> select the delete blob and Undelete it.)
3. Note down before/after details (uploaded blob name, delete time, recovery visibility).

### Cleanup
1. Permanently remove temporary test blobs/versions used in this check.
2. Keep baseline storage account configuration unchanged.

## IaC Templates (Optional - Later)
- ARM: `labs/arm/day1/scenario02/main.json`

## Prerequisites
- `rg-az104` exists in `northeurope`
- Unique storage account name selected (3-24 lowercase alphanumeric)
- If missing, create it with Azure CLI: `az group create --name rg-az104 --location northeurope --output table`

## Sequence Notes
- Primary path: Complete the Portal steps first; use CLI helper commands for verification and troubleshooting.
- If a command fails: wait 1-2 minutes and retry once (control-plane/data-plane propagation delays are common).
- If still blocked: validate in Portal, note down the blocker, and continue to the next lab step.

## Quick Run (Azure CLI)
```bash
RG=rg-az104
LOC=northeurope
SA=staz104$RANDOM$RANDOM
az account show --output table
az group show --name $RG --output table
az deployment group create --resource-group $RG --template-file labs/arm/day1/scenario02/main.json --parameters location=$LOC storageAccountName=$SA containerName=labdata deleteRetentionDays=7
az storage account show --name $SA --resource-group $RG --output table
az storage container list --account-name $SA --auth-mode login --output table
```

### One-Liner (copy/paste)
```bash
RG=rg-az104 && LOC=northeurope && SA=staz104$RANDOM$RANDOM && az account show --output table && az group create --name $RG --location $LOC --output table && az group show --name $RG --output table && az deployment group create --resource-group $RG --template-file labs/arm/day1/scenario02/main.json --parameters location=$LOC storageAccountName=$SA containerName=labdata deleteRetentionDays=7 && az storage account show --name $SA --resource-group $RG --output table && az storage container list --account-name $SA --auth-mode login --output table
```

## Exercise
1. Set variables:
   - `RG=rg-az104`
   - `LOC=northeurope`
   - `SA=staz104$RANDOM$RANDOM`
2. Deploy with ARM template (recommended):
   - `az deployment group create --resource-group $RG --template-file labs/arm/day1/scenario02/main.json --parameters location=$LOC storageAccountName=$SA containerName=labdata deleteRetentionDays=7`
3. Or deploy with ARM:
   - `az deployment group create --resource-group $RG --template-file labs/arm/day1/scenario02/main.json --parameters location=$LOC storageAccountName=$SA containerName=labdata deleteRetentionDays=7`
4. Optional post-deploy check:
   - `az storage account show --name $SA --resource-group $RG --output table`
   - `az storage container list --account-name $SA --auth-mode login --output table`

## Portal Path (Alternative)
1. Open Azure Portal and go to **Resource groups** > **rg-az104**.
2. Open **Deployments** and verify the Day 1 Lab 2 deployment status is **Succeeded**.
3. Open the deployed storage account resource and review **Overview** (location and SKU).
4. In the storage account, open **Data storage** > **Containers** and verify `labdata` exists.
5. Open **Data protection** and confirm blob soft delete/versioning settings match lab expectations.

## Validation
- Storage account deployed in North Europe.
- Blob versioning and soft delete are enabled.
- `labdata` container exists.

## Complete Cleanup
1. Delete container data (optional):
   - `az storage container delete --name labdata --account-name $SA --auth-mode login`
2. Delete storage account:
   - `az storage account delete --name $SA --resource-group $RG --yes`
