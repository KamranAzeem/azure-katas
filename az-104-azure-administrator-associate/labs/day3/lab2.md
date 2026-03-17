# Day 3 - Lab 2: Azure Files Baseline

## Context (Why this lab matters)
- Azure Files is a common AZ-104 storage workload with protocol and access-tier decisions.
- This lab reinforces file-share creation and SMB baseline validation.
- You should finish with one verified share and reproducible checks.

## Objective
Create Azure Files share and validate SMB baseline.

## Scenario
An application team needs a simple shared file endpoint with known baseline settings.
You create `fileshare01` and verify expected tier/protocol behavior.

## Direct Portal GUI Steps (Primary)
1. Open Azure Portal -> **Storage accounts** -> select existing account in `rg-az104`.
   - Recommended: reuse Day 3 Lab 1 account (for example `staz104d30303`).
   - If no account exists, create a new one in `rg-az104`.
2. In the storage account, open **Data storage** -> **File shares** -> **+ File share**.
3. In **Basics** tab, set:
   - Name: `fileshare01`
   - Access tier: `Transaction optimized`
4. In **Backup** tab:
   - Keep **Enabled backup** disabled (unchecked).
5. Create the share.
6. Open the created share and confirm:
   - Access tier is `Transaction optimized`
   - Maximum capacity is shown as a preset value (typically `100 TiB`) in the Portal.
7. Optional/conditional check:
   - If this is a newly created storage account for Lab 2, quickly verify in **Networking** and **Configuration** that baseline security settings are correct.
   - If you reused the Day 3 Lab 1 storage account, you can skip this step because account-level security baseline was already validated there.
8. Note down storage account name, share name, and access tier for validation.

## Portal Notes
- You may not see an explicit **SMB** selector in the create-share flow; for this lab path, SMB is still the effective protocol (verify with CLI `EnabledProtocols`).
- You may not be able to set a custom share size in this Portal screen; capacity is shown as a preset maximum (commonly `100 TiB`).

## Azure CLI Helper
```bash
RG=rg-az104
az account show --output table
az group show --name $RG --output table
az storage account list --resource-group $RG --query "[].{Name:name,Location:location}" --output table
# Replace <storage-account-name> with your account
az storage share-rm list --resource-group $RG --storage-account <storage-account-name> --query "[].{Name:name,AccessTier:accessTier,EnabledProtocols:enabledProtocols}" --output table
```

## Optional Deep-Dive Exercise (SMB Access Proof)
Use this only if you want observable share access behavior.

### Create
1. Upload one small test file into `fileshare01` using Portal File share browser.
2. If you have a lab VM, mount the share (SMB) from that VM using storage account credentials.

### Validate
1. Confirm file is visible in Portal and from mounted path (if mounted).
2. Re-run share baseline query:
   - `az storage share-rm list --resource-group rg-az104 --storage-account <storage-account-name> --query "[].{Name:name,AccessTier:accessTier,EnabledProtocols:enabledProtocols}" --output table`
3. Note down details (file name, protocol, access tier).

### Cleanup
1. Remove temporary test file and unmount share (if mounted).
2. Keep `fileshare01` baseline unless main cleanup step requires deletion.

## IaC Templates (Optional - Later)
- ARM: `labs/arm/day3/scenario02/main.json`

## Prerequisites
- `rg-az104` in `northeurope`
- If missing, create it with Azure CLI: `az group create --name rg-az104 --location northeurope --output table`

## Sequence Notes
- Primary path: Complete the Portal steps first; use CLI helper commands for verification and troubleshooting.
- If continuing from Day 3 Lab 1, reuse the same storage account and only add the file share in this lab.
- When reusing Day 3 Lab 1 account, focus Lab 2 validation on file-share settings (name/tier/protocol), not repeating account-level hardening checks.
- If a command fails: wait 1-2 minutes and retry once (control-plane/data-plane propagation delays are common).
- If still blocked: validate in Portal, note down the blocker, and continue to the next lab step.

## Quick Run (Azure CLI)
```bash
RG=rg-az104
LOC=northeurope
SA=staz104file$RANDOM
az account show --output table
az group show --name $RG --output table
az deployment group create --resource-group $RG --template-file labs/arm/day3/scenario02/main.json --parameters location=$LOC storageAccountName=$SA fileShareName=fileshare01 shareQuotaGiB=100
az storage share-rm list --resource-group $RG --storage-account $SA --output table
```

### One-Liner (copy/paste)
```bash
RG=rg-az104 && LOC=northeurope && SA=staz104file$RANDOM && az account show --output table && az group create --name $RG --location $LOC --output table && az group show --name $RG --output table && az deployment group create --resource-group $RG --template-file labs/arm/day3/scenario02/main.json --parameters location=$LOC storageAccountName=$SA fileShareName=fileshare01 shareQuotaGiB=100 && az storage share-rm list --resource-group $RG --storage-account $SA --output table
```

## Exercise
1. Set variables:
   - `RG=rg-az104`
   - `LOC=northeurope`
   - `SA=staz104file$RANDOM`
2. Deploy with ARM template (recommended):
   - `az deployment group create --resource-group $RG --template-file labs/arm/day3/scenario02/main.json --parameters location=$LOC storageAccountName=$SA fileShareName=fileshare01 shareQuotaGiB=100`
3. Or deploy with ARM:
   - `az deployment group create --resource-group $RG --template-file labs/arm/day3/scenario02/main.json --parameters location=$LOC storageAccountName=$SA fileShareName=fileshare01 shareQuotaGiB=100`
4. List shares:
   - `az storage share-rm list --resource-group $RG --storage-account $SA --query "[].{Name:name,AccessTier:accessTier,EnabledProtocols:enabledProtocols}" --output table`

## Validation
- File share is created with expected name and access tier (`Transaction optimized`).

## Troubleshooting
- If `az storage share-rm list` returns no rows, the share was not created (or not yet visible).
- Portal fix:
   1. Open storage account -> **File shares**.
   2. Create/save `fileshare01` with access tier `Transaction optimized`.
   3. Wait 1-2 minutes and re-run validation.
- CLI fallback (replace `<storage-account-name>`):
   - `az storage share-rm create --resource-group rg-az104 --storage-account <storage-account-name> --name fileshare01 --quota 100 --enabled-protocols SMB --output table`
- Re-run validation:
   - `az storage share-rm list --resource-group rg-az104 --storage-account <storage-account-name> --query "[].{Name:name,AccessTier:accessTier,EnabledProtocols:enabledProtocols}" --output table`

## Complete Cleanup
- If you reused an existing storage account (recommended):
   - `az storage share-rm delete --resource-group rg-az104 --storage-account <storage-account-name> --name fileshare01 --yes`
- If you created a dedicated lab storage account just for this exercise:
   - `az storage account delete --name $SA --resource-group rg-az104 --yes`
