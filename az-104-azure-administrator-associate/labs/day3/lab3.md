# Day 3 - Lab 3: Backup Starter

## Context (Why this lab matters)
- Backup configuration and recovery readiness are core AZ-104 operational outcomes.
- This lab introduces vault setup and backup-object verification.
- You should finish with a validated backup baseline and clear cleanup path.

## Objective
Create backup vault baseline and policy objects.

## Scenario
Your team needs a backup-ready baseline before enabling workload protection.
You create the vault, review backup settings, and capture readiness details.

## Direct Portal GUI Steps (Primary)
1. Open Azure Portal -> **Recovery Services vaults** -> **Create**.
2. Use `rg-az104`, region `North Europe`, and vault name `rsv-az104-d3`.
3. Open the created vault and review settings in:
   - **Settings** -> **Networking** (public/private network access controls)
   - **Settings** -> **Properties** (vault basics)
   - **Manage** -> **Backup Infrastructure** and **Overview/Backup** (backup-related configuration views)
4. (Optional) Adjust storage redundancy for practice.
5. Note down vault name and resulting redundancy settings.

## Portal Notes
- Baseline for this lab: keep vault **Public network access** as `Enabled`.
- Baseline for this lab: treat **Backup storage redundancy** as optional unless your step explicitly asks to change it.
- For redundancy in Portal, check **Settings** -> **Properties** for **Backup Configuration** / **Backup storage redundancy**.
- In some portal versions, network access appears in **Settings** -> **Networking** instead of **Properties**.
- Immediately after vault creation, some backup/storage settings may take a short time to become editable or visible.
- If a setting seems missing, use the vault search box (for example search `redundancy`) and retry after 1-2 minutes before treating it as a blocker.

## Azure CLI Helper
```bash
RG=rg-az104
az account show --output table
az group show --name $RG --output table
az backup vault list --resource-group $RG --query "[].{Name:name,Location:location}" --output table
az backup vault show --name rsv-az104-d3 --resource-group $RG --output table
```

## Optional Deep-Dive Exercise (Backup Job Proof)
Use this only if you want explicit backup execution details.

### Create
1. Reuse a file share workload (for example from Day 3 Lab 2) and enable backup in `rsv-az104-d3`.
2. Trigger one **Backup now** operation:
   - Open `rsv-az104-d3` -> **Protected items** -> **Backup items** -> **Azure Storage (Azure Files)** -> select `fileshare01` -> **Backup now**.
   - If **Backup now** is not visible on Overview, open the protected item blade first (or use **Backup jobs** -> **+ Backup now** if shown in your portal version).

### Validate
1. In Recovery Services Vault, open Monitoring -> **Backup jobs** and confirm job state transitions.
2. Confirm protected item appears under vault protected items.
3. Note down details: protected item name, job ID/status, timestamp.

### Recovery / Reset
1. Stop protection for the temporary protected item (retain data or delete per your test intent).
2. Confirm no new scheduled jobs are created for the removed test item.

### Cleanup
1. Remove any temporary backup item created only for this optional test.
2. Keep core vault baseline unless executing full lab cleanup.

## IaC Templates (Optional - Later)
- ARM: `labs/arm/day3/scenario03/main.json`

## Prerequisites
- `rg-az104` in `northeurope`
- If missing, create it with Azure CLI: `az group create --name rg-az104 --location northeurope --output table`

## Sequence Notes
- Primary path: Complete the Portal steps first; use CLI helper commands for verification and troubleshooting.
- If a command fails: wait 1-2 minutes and retry once (control-plane/data-plane propagation delays are common).
- If still blocked: validate in Portal, note down the blocker, and continue to the next lab step.

## Quick Run (Azure CLI)
```bash
RG=rg-az104
LOC=northeurope
RSV=rsv-az104-d3
az account show --output table
az group show --name $RG --output table
az deployment group create --resource-group $RG --template-file labs/arm/day3/scenario03/main.json --parameters location=$LOC recoveryServicesVaultName=$RSV
az backup vault show --name $RSV --resource-group $RG --output table
```

### One-Liner (copy/paste)
```bash
RG=rg-az104 && LOC=northeurope && RSV=rsv-az104-d3 && az account show --output table && az group create --name $RG --location $LOC --output table && az group show --name $RG --output table && az deployment group create --resource-group $RG --template-file labs/arm/day3/scenario03/main.json --parameters location=$LOC recoveryServicesVaultName=$RSV && az backup vault show --name $RSV --resource-group $RG --output table
```

## Exercise
1. Set variables:
   - `RG=rg-az104`
   - `LOC=northeurope`
   - `RSV=rsv-az104-d3`
2. Deploy with ARM template (recommended):
   - `az deployment group create --resource-group $RG --template-file labs/arm/day3/scenario03/main.json --parameters location=$LOC recoveryServicesVaultName=$RSV`
3. Or deploy with ARM:
   - `az deployment group create --resource-group $RG --template-file labs/arm/day3/scenario03/main.json --parameters location=$LOC recoveryServicesVaultName=$RSV`
4. Optional: set redundancy for practice:
   - `az backup vault backup-properties set --name $RSV --resource-group $RG --backup-storage-redundancy LocallyRedundant`
5. Review vault properties:
   - `az backup vault show --name $RSV --resource-group $RG --output table`

## Validation
- Vault exists and redundancy settings are applied.

## Optional Mini Exercise (Use the Vault for Real)
Goal: protect an existing Azure File share using this vault with minimal/no new resources.

1. Reuse Lab 2 resources:
   - Storage account: `staz104d30303`
   - File share: `fileshare01`
2. In `rsv-az104-d3`, open **Backup** -> **+ Backup**.validate now
3. Set **Where is your workload running?** to `Azure` and **What do you want to back up?** to `Azure File Share`.
4. Select storage account `staz104d30303`, then choose share `fileshare01`.
5. Use default policy (or create a simple daily policy), then enable backup.
6. Trigger **Backup now** once, and confirm a job appears in **Backup Jobs**.
   - Path: `rsv-az104-d3` -> **Backup items** -> **Azure Storage (Azure Files)** -> `fileshare01` -> **Backup now**.

Minimal validation for this optional exercise:
- Protected items show the file share under the vault.
- At least one backup job is visible (In progress or Completed).

If Lab 2 resources were deleted, either skip this optional exercise or create one small file share first.

## Complete Cleanup
Use this exact sequence to avoid delete failures from lingering associations:

1. Open vault `rsv-az104-d3` -> **Backup items**.
   - For each protected item (Azure File Share / VM / other): select item -> **Stop backup**.
   - If this is lab-only data, choose **Delete backup data** when prompted.

2. Open **Backup Infrastructure** in the same vault.
   - Under **Storage Accounts**, unregister associated storage accounts.
   - Under **Protected Servers** (if any), unregister associated servers.
   - Remove/unregister any remaining protection containers if shown.

3. Open **Backup jobs** and wait until stop/unregister operations are completed.
   - Do not proceed while jobs are still running.

4. Try vault deletion:
   - `az backup vault delete --name rsv-az104-d3 --resource-group rg-az104 --yes`

5. If deletion is blocked by soft-delete/security settings in your tenant:
   - In vault **Properties/Security settings**, review soft-delete state and follow your org policy.
   - After clearing blocking protected items/containers, retry vault deletion.

6. Final validation:
   - `az backup vault list --resource-group rg-az104 --query "[?name=='rsv-az104-d3']" --output table`
   - Expected: no rows returned.
