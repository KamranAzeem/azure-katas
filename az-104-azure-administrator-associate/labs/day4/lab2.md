# Day 4 - Lab 2: Monitoring and Alerts

## Context (Why this lab matters)
- Monitoring and alert routing are frequent AZ-104 operational responsibilities.
- This lab practices workspace + action-group + alert-rule wiring with validation details.
- You should finish with one end-to-end alert path you can troubleshoot confidently.

## Objective
Create a Log Analytics workspace, create an alert action group, and create one simple alert rule linked to that action group.

## Scenario
You need a minimal monitoring pipeline for operations alerting.
You wire workspace, alert logic, and action group, then verify end-to-end linkage.

## Direct Portal GUI Steps (Primary)
1. Pre-check resource provider registration in Portal (recommended before creating LAW):
   - Open **Subscriptions** -> select your active subscription.
   - Open **Resource providers**.
   - Search for `Microsoft.Insights`.
   - If status is not `Registered`, select it and click **Register**.
   - Wait until status shows `Registered`.
2. Open Azure Portal -> **Log Analytics workspaces** -> **Create**.
3. Use `rg-az104`, `North Europe`, and workspace name `law-az104-d4`.
4. Create action group (separate flow):
   - Open **Monitoring** -> **Alerts** -> **Action groups** -> **Create**
   - Name: `ag-az104-d4`
   - Short name: `ag104d4`
   - Resource group: `rg-az104`
   - Leave notification channels empty for this baseline lab, then create.
5. Create alert rule (separate flow):
   - Open **Monitoring** -> **Alerts** -> **Alert rules** -> **Create**
   - Scope: `law-az104-d4` (Log Analytics workspace)
   - Condition:
     - Signal name: `Custom log search` (sometimes shown as `Log search`)
     - Search query (KQL): `print 1`
     - Alert logic: `Number of results` `Greater than` `0`
     - Evaluation frequency / Period: `5 minutes` / `5 minutes`
   - Actions: select action group `ag-az104-d4`
   - Alert rule name: `ar-law-az104-d4`
   - Severity: `3` (or default)
   - Create the rule.
6. Note down workspace, action group, and alert rule names.

## Portal Notes
- Action groups are **not** created inside the alert-rule wizard tabs first; they are managed under **Alerts -> Action groups**.
- If your selected workspace signal is not available yet, wait 1-2 minutes and refresh.
- Fallback: create any simple alert rule that successfully binds `ag-az104-d4`; the key outcome is understanding scope + condition + action group wiring.

## Azure CLI Helper
```bash
RG=rg-az104
az account show --output table
az monitor log-analytics workspace list --resource-group $RG --output table
az monitor action-group list --resource-group $RG --output table
```

## Optional Deep-Dive Exercise (Proof of Impact)
Use this only if you want observable alert details beyond simple resource creation.

### Create
1. Open **Monitoring** -> **Alerts** -> **Alert rules** and open `ar-law-az104-d4`.
2. Confirm scope is `law-az104-d4` and action group is `ag-az104-d4`.
3. Force a deterministic match using the existing query path (`print 1`) with evaluation frequency/period set to `5 minutes`.

### Validate
1. Wait one evaluation cycle (typically 5-10 minutes).
2. In **Monitoring** -> **Alerts** -> **Alert rules** -> `ar-law-az104-d4`, review **Alert instances**.
   - Expected: at least one alert instance appears as fired/resolved history depending on timing.
3. Confirm action-group linkage details:
   - Open the alert rule details and verify `ag-az104-d4` is listed under **Actions**.
4. Capture details in your notes:
   - Rule name, evaluation timestamp, instance status, and linked action group name.

### Recovery / Reset
1. Edit the alert rule query to a non-triggering value:
   - `print 0 | where 1 == 2`
2. Save and wait one evaluation cycle.
3. Confirm no new fired instances are created after the reset point.

### Cleanup
1. If you created additional test alert rules for this deep dive, delete those extra rules.
2. Keep baseline resources for the main lab path unless you are executing full lab cleanup.

## IaC Templates (Optional - Later)
- ARM: `labs/arm/day4/scenario02/main.json`

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
LAW=law-az104-d4
AG=ag-az104-d4
az account show --output table
az group show --name $RG --output table
az deployment group create --resource-group $RG --template-file labs/arm/day4/scenario02/main.json --parameters location=$LOC workspaceName=$LAW actionGroupName=$AG actionGroupShortName=ag104d4
az monitor log-analytics workspace show --resource-group $RG --workspace-name $LAW --output table
```

### One-Liner (copy/paste)
```bash
RG=rg-az104 && LOC=northeurope && LAW=law-az104-d4 && AG=ag-az104-d4 && az account show --output table && az group create --name $RG --location $LOC --output table && az group show --name $RG --output table && az deployment group create --resource-group $RG --template-file labs/arm/day4/scenario02/main.json --parameters location=$LOC workspaceName=$LAW actionGroupName=$AG actionGroupShortName=ag104d4 && az monitor log-analytics workspace show --resource-group $RG --workspace-name $LAW --output table
```

## Exercise
1. Set variables:
   - `RG=rg-az104`
   - `LOC=northeurope`
2. Deploy with ARM template (recommended):
   - `az deployment group create --resource-group $RG --template-file labs/arm/day4/scenario02/main.json --parameters location=$LOC workspaceName=law-az104-d4 actionGroupName=ag-az104-d4 actionGroupShortName=ag104d4`
3. Or deploy with ARM:
   - `az deployment group create --resource-group $RG --template-file labs/arm/day4/scenario02/main.json --parameters location=$LOC workspaceName=law-az104-d4 actionGroupName=ag-az104-d4 actionGroupShortName=ag104d4`
4. Review monitoring assets:
   - `az monitor log-analytics workspace show --resource-group $RG --workspace-name law-az104-d4 --output table`

## Validation
- Workspace and action group are deployed.
- Optional deep-dive path: alert instance history and action-group linkage details are captured, then reset behavior is verified.

## Complete Cleanup
- `az monitor scheduled-query delete --resource-group rg-az104 --name ar-law-az104-d4 --yes`
- `az monitor action-group delete --resource-group rg-az104 --name ag-az104-d4`
- `az monitor log-analytics workspace delete --resource-group rg-az104 --workspace-name law-az104-d4 --yes`

Cleanup verification (optional):
- `az monitor scheduled-query list --resource-group rg-az104 --output table`
