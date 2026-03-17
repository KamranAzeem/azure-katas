# Day 4 - Lab 3: Automation Starter

Azure Automation Account is a managed service for operational automation in Azure.
It hosts runbooks (scripts/workflows), supports scheduling/on-demand execution, and can use managed identity for secure access to Azure resources.
Common use cases include start/stop schedules, housekeeping, and repeatable operational tasks.

## Context (Why this lab matters)
- AZ-104 includes automation readiness and operational repeatability.
- This lab establishes Automation Account baseline health and runbook readiness checks.
- You should finish with a verified automation foundation and simple validation details.

## Objective
Create an Automation Account in `rg-az104`, verify core configuration (identity + runbook readiness), and confirm it is ready for basic automation operations.

## Scenario
You are preparing automation capabilities for routine operations.
You must validate account readiness, identity availability, and basic runbook path visibility.

## Direct Portal GUI Steps (Primary)
1. Open Azure Portal -> **Automation Accounts** -> **Create**.
2. In **Basics** tab, set:
   - Subscription: your active lab subscription
   - Resource group: `rg-az104`
   - Automation account name: `aa-az104-d4`
   - Region: `North Europe`
3. Keep remaining tabs/settings at baseline defaults and click **Review + create** -> **Create**.
4. Open `aa-az104-d4` and review:
   - **Account Settings** -> **Identity** (confirm status and object details page loads)
   - **Process Automation** -> **Runbooks** (confirm runbook workspace is available)
5. (Optional) In **Runbooks**, create/import a simple runbook draft for familiarization.
6. Note down account name, region, and readiness state for runbook operations.

## Portal Notes
- If **Automation Accounts** is not visible immediately, use global search in Portal for `Automation Accounts`.
- First-time open can take 1-2 minutes to fully load identity/runbook blades after account creation.
- Identity validation note: seeing **System assigned = ON** is an expected and valid outcome for this lab.

## Azure CLI Helper
```bash
RG=rg-az104
az account show --output table
az automation account list --resource-group $RG --output table
az automation account show --resource-group $RG --name aa-az104-d4 --output table
```

## Optional Deep-Dive Exercise (Runbook Execution Proof)
Use this only if you want one concrete automation execution signal.

### Create
1. In `aa-az104-d4`, create a simple PowerShell runbook (for example `rb-az104-hello`).
2. Add a minimal script (for example output current date/time) and publish the runbook.

### Validate
1. Start the runbook manually in Portal.
2. Confirm job status changes to `Completed` and output is visible.
3. Note down details: runbook name, job ID, completion timestamp.

### Cleanup
1. Delete temporary runbook created only for this optional check.
2. Keep automation account baseline unless full lab cleanup is being performed.

## IaC Templates (Optional - Later)
- ARM: `labs/arm/day4/scenario03/main.json`

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
AA=aa-az104-d4
az account show --output table
az group show --name $RG --output table
az deployment group create --resource-group $RG --template-file labs/arm/day4/scenario03/main.json --parameters location=$LOC automationAccountName=$AA
az automation account show --resource-group $RG --name $AA --output table
```

### One-Liner (copy/paste)
```bash
RG=rg-az104 && LOC=northeurope && AA=aa-az104-d4 && az account show --output table && az group create --name $RG --location $LOC --output table && az group show --name $RG --output table && az deployment group create --resource-group $RG --template-file labs/arm/day4/scenario03/main.json --parameters location=$LOC automationAccountName=$AA && az automation account show --resource-group $RG --name $AA --output table
```

## Exercise
1. Set variables:
   - `RG=rg-az104`
   - `LOC=northeurope`
   - `AA=aa-az104-d4`
2. Deploy with ARM template (recommended):
   - `az deployment group create --resource-group $RG --template-file labs/arm/day4/scenario03/main.json --parameters location=$LOC automationAccountName=$AA`
3. Or deploy with ARM:
   - `az deployment group create --resource-group $RG --template-file labs/arm/day4/scenario03/main.json --parameters location=$LOC automationAccountName=$AA`
4. Verify account:
   - `az automation account show --resource-group $RG --name $AA --output table`
5. Optional: import runbook modules in Portal.

## Validation
- Automation account exists and is ready for runbooks.

## Complete Cleanup
- `az automation account delete --resource-group rg-az104 --name aa-az104-d4 --yes`
