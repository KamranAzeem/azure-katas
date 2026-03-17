# Day 5 - Lab 1: Timed Practical Round 1 (Mixed-Domain Baseline)

## Context (Why this lab matters)
- This is Round 1 of Day 5 simulation and should be run with a time box.
- You validate mixed-domain baseline operations quickly (deployment, governance, and monitoring details).
- Use it to measure speed, accuracy, and details quality under exam-like pressure.

## Objective
Complete a timed mixed-domain validation in `rg-az104` and capture details with minimal rework.

## Scenario
You are in a timed practical round and must prove baseline admin readiness quickly.
You deploy/verify core resources and capture governance + activity details in one pass.

## Microsoft Learn Alignment Note
- This lab is AZ-104 objective-aligned, but it is not a verbatim copy of a single Microsoft Learn guided exercise.
- It follows the same skill areas (resource deployment, governance/tag checks, monitoring details, and validation), adapted into one consolidated review flow.
- You can treat it as an exam-practice wrap-up scenario; equivalent alternatives are domain-by-domain Learn modules or troubleshooting-focused mini labs.

## Direct Portal GUI Steps (Primary)
1. Open Azure Portal -> **Resource groups** -> **rg-az104**.
2. Open **Overview** and confirm baseline state:
   - Resource group exists in `northeurope`.
   - At least expected baseline resources from previous days are visible.
3. Open **Tags** and verify governance tags are present.
   - Existing baseline tags should include: `Environment=Lab`, `Course=AZ-104`, `Owner=kamran`.
4. Open **Deployments** under the resource group and check latest deployment status.
   - If no Day 5 deployment exists yet, continue to Step 5.
5. Create/confirm final-review storage account:
   - Go to **Storage accounts** -> **+ Create**.
   - Resource group: `rg-az104`
   - Storage account name: use unique lowercase name (for example `staz104d5<unique>`)
   - Region: `North Europe`
   - Performance: `Standard`
   - Redundancy: `LRS`
   - Click **Review + create** -> **Create**.
6. Create/confirm Log Analytics workspace:
   - Go to **Log Analytics workspaces** -> **+ Create**.
   - Resource group: `rg-az104`
   - Name: `law-az104-d5`
   - Region: `North Europe`
   - Pricing tier: `PerGB2018` (default in most tenants) (not selectable / not visible)
   - Retention: `30 days`
   - Click **Review + create** -> **Create**.
7. Back in **Resource groups** -> **rg-az104**, open **Activity log**.
   - Filter to recent operations and confirm successful create/update operations for storage and workspace.
8. Note down final validation snapshot (minimum):
   - Resource inventory contains Day 5 storage + workspace.
   - RG tag posture (existing baseline tags plus Day 5 marker tag if applied, currently `FinalReview=true` via template).
   - Activity log shows successful deployment operations in this session.

## Azure CLI Helper
```bash
RG=rg-az104
az account show --output table
az group show --name $RG --query tags --output table
az resource list --resource-group $RG --output table
az monitor log-analytics workspace list --resource-group $RG --output table
```

## Optional Deep-Dive Exercise (Final-Review Drift Proof)
Use this only if you want a controlled drift-and-reconcile practice.

### Create
1. Deploy Day 5 baseline resources.
2. Introduce one controlled drift in Portal (for example update/remove one non-critical RG tag).

### Validate
1. Confirm drift exists via:
   - `az group show --name rg-az104 --query tags --output table`
2. Review Activity Log details for the change operation.
3. Note down before/after tag posture.

### Recovery / Reset
1. Re-apply expected baseline tags (or redeploy scenario01 template).
2. Confirm intended tag posture and resource inventory are restored.

### Cleanup
1. Remove only temporary drift-test changes.
2. Keep baseline resources for continuation to Lab 2 unless you are performing full Day 5 cleanup.

## IaC Templates (Optional - Later)
- ARM: `labs/arm/day5/scenario01/main.json`

## Prerequisites
- `rg-az104` in `northeurope`
- Prior day templates available
- If missing, create it with Azure CLI: `az group create --name rg-az104 --location northeurope --output table`

## Sequence Notes
- Primary path: Complete the Portal steps first; use CLI helper commands for verification and troubleshooting.
- If a command fails: wait 1-2 minutes and retry once (control-plane/data-plane propagation delays are common).
- If still blocked: validate in Portal, note down the blocker, and continue to the next lab step.

## Quick Run (Azure CLI)
```bash
RG=rg-az104
LOC=northeurope
SA=staz104d5$RANDOM
LAW=law-az104-d5
az account show --output table
az group show --name $RG --output table
az deployment group create --resource-group $RG --template-file labs/arm/day5/scenario01/main.json --parameters location=$LOC storageAccountName=$SA workspaceName=$LAW
az resource list --resource-group $RG --output table
```

### One-Liner (copy/paste)
```bash
RG=rg-az104 && LOC=northeurope && SA=staz104d5$RANDOM && LAW=law-az104-d5 && az account show --output table && az group create --name $RG --location $LOC --output table && az group show --name $RG --output table && az deployment group create --resource-group $RG --template-file labs/arm/day5/scenario01/main.json --parameters location=$LOC storageAccountName=$SA workspaceName=$LAW && az resource list --resource-group $RG --output table
```

## Exercise
1. Set variables:
   - `RG=rg-az104`
   - `LOC=northeurope`
   - `SA=staz104d5$RANDOM`
   - `LAW=law-az104-d5`
2. Deploy with ARM template (recommended):
   - `az deployment group create --resource-group $RG --template-file labs/arm/day5/scenario01/main.json --parameters location=$LOC storageAccountName=$SA workspaceName=$LAW`
3. Or deploy with ARM:
   - `az deployment group create --resource-group $RG --template-file labs/arm/day5/scenario01/main.json --parameters location=$LOC storageAccountName=$SA workspaceName=$LAW`
4. Validate resource inventory:
   - `az resource list --resource-group $RG --output table`
5. Validate tags and role visibility at RG scope.

## Validation
- Deployment completes successfully at RG scope (Portal **Deployments** shows `Succeeded` for the latest Day 5 final-review deployment).
- Resource inventory in `rg-az104` includes both expected Day 5 baseline resources:
   - Storage account created in this lab
   - Log Analytics workspace `law-az104-d5`
- RG governance posture is visible and correct for this lab run (baseline tags remain present; Day 5 marker tag visible if applied, currently `FinalReview=true` via template).
- Activity log contains successful create/update operations corresponding to this lab session.
- Final note is recorded with: deployed resources, tag posture, and deployment/activity-log confirmation.

## Complete Cleanup
- `az storage account delete --resource-group rg-az104 --name $SA --yes`
- `az monitor log-analytics workspace delete --resource-group rg-az104 --workspace-name $LAW --yes`
- `az group update --name rg-az104 --remove tags.FinalReview`
