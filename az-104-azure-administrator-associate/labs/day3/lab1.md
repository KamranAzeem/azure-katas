# Day 3 - Lab 1: Storage Security + Lifecycle

## Context (Why this lab matters)
- Day-to-day AZ-104 storage operations include security posture and lifecycle governance.
- This lab combines secure account defaults with lifecycle policy validation.
- You should finish with policy-backed storage controls and clear details.

## Objective
Deploy storage with lifecycle policy and security defaults.

## Scenario
You are asked to harden a storage account and enforce lifecycle retention policy.
You must show both security baseline settings and policy configuration details.

## Direct Portal GUI Steps (Primary)
1. Open Azure Portal -> **Storage accounts** -> **Create** in `rg-az104`.
2. In **Basics** tab, set:
   - Resource group: `rg-az104`
   - Region: `North Europe`
   - Storage account name: unique valid name (e.g. staz104d30303)
   - Performance: `Standard`
   - Redundancy: `LRS`
3. In **Advanced** tab, under **Blob storage**, set **Access tier** to `Hot`.
4. In **Networking** tab, set **Public network access** to `Disable`.
5. In **Data protection** tab:
   - Enable soft delete for blobs, containers, and file shares
   - Set days to retain deleted blobs to `7`
6. Create the storage account, then open the new account and confirm baseline security settings.
7. Open **Lifecycle management** and create this rule:
   - Rule name: `move_to_cool_and_delete_after_a_year`
   - Scope/filters: apply to all block blobs (no prefix filter)
   - Action 1: move blob to cool tier when **days after last modification** is greater than `30`
   - Action 2: delete blob when **days after last modification** is greater than `365`
8. Save policy (with this rule in it) and confirm it appears in the account policy view.
9. Note down rule names and day thresholds.

## Portal Notes
- The Lifecycle rule must be explicitly saved before CLI validation; otherwise `ManagementPolicyNotFound` is expected.
- Portal labels can vary slightly by blade version (for example, where **Lifecycle management** appears), so use account search if needed.

## Azure CLI Helper
```bash
RG=rg-az104
az account show --output table
az group show --name $RG --output table
az storage account list --resource-group $RG --query "[].{Name:name,Location:location}" --output table
# Replace <storage-account-name> with your account
az storage account management-policy show --resource-group $RG --account-name <storage-account-name> --output json
```

**Note:**
The last command above will only shows policy metadata:
* "Name" is always "DefaultManagementPolicy" (the policy object name)
* It does not show individual lifecycle rule names in table view

## Optional Deep-Dive Exercise (Lifecycle Policy Proof)
Use this only if you want practical details that policy configuration is active.

### Create
1. Create a temporary container (for example `lifecycletest`) and upload one test blob.
2. Confirm management policy JSON includes your expected cool/delete actions.

### Validate
1. Run:
   - `az storage account management-policy show --resource-group rg-az104 --account-name <storage-account-name> --output json`
2. Confirm rule name, filters, and action thresholds are present.
3. Note down details (policy snippet + blob metadata baseline).

### Cleanup
1. Delete temporary test blob/container created for this optional check.
2. Keep the baseline lifecycle policy in place for the main lab objective.

## IaC Templates (Optional - Later)
- ARM: `labs/arm/day3/scenario01/main.json`

## Prerequisites
- `rg-az104` in `northeurope`
- If missing, create it with Azure CLI: `az group create --name rg-az104 --location northeurope --output table`

## Sequence Notes
- Primary path: Complete the Portal steps first; use CLI helper commands for verification and troubleshooting.
- Save the lifecycle management rule before running `az storage account management-policy show`; otherwise `ManagementPolicyNotFound` is expected.
- If you are continuing directly to Day 3 Lab 2, keep this storage account and reuse it for Azure Files to save time.
- If a command fails: wait 1-2 minutes and retry once (control-plane/data-plane propagation delays are common).
- If still blocked: validate in Portal, note down the blocker, and continue to the next lab step.

## Quick Run (Azure CLI)
```bash
# Set this (NAME_SUFFIX) once only and use this throughout the lab. 
#   Otherwise you get a new suffix every time.
NAME_SUFFIX=${RANDOM}
```

```bash
RG=rg-az104
LOC=northeurope

SA=staz104d3${NAME_SUFFIX}

echo "Storage account name is: ${SA}"

az account show --output table
az group show --name $RG --output table
az deployment group create --resource-group $RG --template-file labs/arm/day3/scenario01/main.json --parameters location=$LOC storageAccountName=$SA lifecycleCoolAfterDays=30 lifecycleDeleteAfterDays=365
az storage account management-policy show --resource-group $RG --account-name $SA --output json
```

### One-Liner (copy/paste)
```bash
RG=rg-az104 && LOC=northeurope && SA=staz104d3$RANDOM && az account show --output table && az group create --name $RG --location $LOC --output table && az group show --name $RG --output table && az deployment group create --resource-group $RG --template-file labs/arm/day3/scenario01/main.json --parameters location=$LOC storageAccountName=$SA lifecycleCoolAfterDays=30 lifecycleDeleteAfterDays=365 && az storage account management-policy show --resource-group $RG --account-name $SA --output json
```

## Exercise
1. Set variables:
   - `RG=rg-az104`
   - `LOC=northeurope`
   - `SA=staz104d3$RANDOM`
2. Deploy with ARM template (recommended):
   - `az deployment group create --resource-group $RG --template-file labs/arm/day3/scenario01/main.json --parameters location=$LOC storageAccountName=$SA lifecycleCoolAfterDays=30 lifecycleDeleteAfterDays=365`
3. Or deploy with ARM:
   - `az deployment group create --resource-group $RG --template-file labs/arm/day3/scenario01/main.json --parameters location=$LOC storageAccountName=$SA lifecycleCoolAfterDays=30 lifecycleDeleteAfterDays=365`
4. Validate lifecycle policy:
   - `az storage account management-policy show --resource-group $RG --account-name $SA --output json`

## Validation
- Lifecycle policy exists and storage security baseline is enabled.

## Troubleshooting
- If validation returns `(ManagementPolicyNotFound) No ManagementPolicy found for account ...`, it means the lifecycle rule has not been created/saved yet.
- Portal fix:
   1. Open the storage account -> **Lifecycle management**.
   2. Add at least one rule (for example: move to Cool after `30` days, delete after `365` days).
   3. Click **Save** and retry validation command.
- CLI fallback (replace `<storage-account-name>`):
   - `az storage account management-policy create --resource-group rg-az104 --account-name <storage-account-name> --policy '{"rules":[{"name":"lifecycle-default","enabled":true,"type":"Lifecycle","definition":{"filters":{"blobTypes":["blockBlob"]},"actions":{"baseBlob":{"tierToCool":{"daysAfterModificationGreaterThan":30},"delete":{"daysAfterModificationGreaterThan":365}}}}}]}'`
- Re-run validation:
   - `az storage account management-policy show --resource-group rg-az104 --account-name <storage-account-name> --output json`

## Complete Cleanup
- Optional now (recommended if continuing to Day 3 Lab 2): keep this storage account and reuse it in Lab 2.
- Cleanup command (use when done with Lab 2 or if you choose not to reuse):
   - `az storage account delete --name $SA --resource-group rg-az104 --yes`
