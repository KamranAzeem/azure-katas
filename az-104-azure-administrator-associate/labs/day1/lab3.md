# Day 1 - Lab 3: Governance + Monitoring Starter

## Context (Why this lab matters)
- AZ-104 operations depend on governance hygiene and quick activity-log interpretation.
- This lab introduces a practical tag-and-monitor loop at resource-group scope.
- You should finish with visible governance details and monitoring signals.

## Objective
Apply tags and review activity/monitoring signals at RG scope.

## Scenario
An ops lead asks for basic governance visibility on a shared resource group.
You apply/verify tags and confirm recent operations through Activity Log details.

## Direct Portal GUI Steps (Primary)
1. Open Azure Portal -> **Resource groups** -> **rg-az104**.
2. Open **Tags** and add/update governance tags (Environment/Course/Day).
3. Open **Activity log** and review recent operations for the RG. (May take few minutes before the "Write tags" activity shows up in the RG's activity log)
4. (Optional) Open **Settings** -> **Policies** and test assignment - if your subscription allows it.
5. Note down observed governance and monitoring signals in the activity log.

## Azure CLI Helper
```bash
RG=rg-az104
az account show --output table
az group show --name $RG --query tags --output table
az monitor activity-log list --resource-group $RG --max-events 20 --output table
az resource list --resource-group $RG --output table
```

## Optional Deep-Dive Exercise (Governance Signal Proof)
Use this only if you want concrete before/after governance details.

### Create
1. Add a temporary RG tag (for example `GovernanceTest=Day1`).
2. Perform one small control-plane update in `rg-az104` (for example update an existing tag value).

### Validate
1. Confirm tag update result via CLI:
   - `az group show --name rg-az104 --query tags --output table`
2. Confirm matching activity-log entries:
   - `az monitor activity-log list --resource-group rg-az104 --max-events 20 --output table`
3. Note down operation name, timestamp, and resulting tag state.

### Cleanup
1. Remove temporary tag used for this optional check.
2. Keep required baseline tags (`Environment`, `Course`, `Day`) as needed.

## IaC Templates (Optional - Later)
- ARM: `labs/arm/day1/scenario03/main.json`

## Prerequisites
- Shared RG: `rg-az104`
- Permission to read activity logs at RG scope
- If missing, create it with Azure CLI: `az group create --name rg-az104 --location northeurope --output table`

## Sequence Notes
- Primary path: Complete the Portal steps first; use CLI helper commands for verification and troubleshooting.
- If a command fails: wait 1-2 minutes and retry once (control-plane/data-plane propagation delays are common).
- If still blocked: validate in Portal, note down the blocker, and continue to the next lab step.

## Quick Run (Azure CLI)
```bash
RG=rg-az104
LOC=northeurope
az account show --output table
az group show --name $RG --output table
az deployment group create --resource-group $RG --template-file labs/arm/day1/scenario03/main.json --parameters location=$LOC environmentTag=Lab courseTag=AZ-104 dayTag=1
az monitor activity-log list --resource-group $RG --max-events 20 --output table
az resource list --resource-group $RG --output table
```

### One-Liner (copy/paste)
```bash
RG=rg-az104 && LOC=northeurope && az account show --output table && az group create --name $RG --location $LOC --output table && az group show --name $RG --output table && az deployment group create --resource-group $RG --template-file labs/arm/day1/scenario03/main.json --parameters location=$LOC environmentTag=Lab courseTag=AZ-104 dayTag=1 && az monitor activity-log list --resource-group $RG --max-events 20 --output table && az resource list --resource-group $RG --output table
```

## Exercise
1. Set variables:
   - `RG=rg-az104`
   - `LOC=northeurope`
2. Deploy governance starter with ARM template (tags only):
   - `az deployment group create --resource-group $RG --template-file labs/arm/day1/scenario03/main.json --parameters location=$LOC environmentTag=Lab courseTag=AZ-104 dayTag=1`
3. Or deploy with ARM (tags only):
   - `az deployment group create --resource-group $RG --template-file labs/arm/day1/scenario03/main.json --parameters location=$LOC environmentTag=Lab courseTag=AZ-104 dayTag=1`
4. Optional: include policy assignment (if permitted):
   - `POLICY_ID=$(az policy definition list --query "[?displayName=='Require a tag and its value on resources'].id | [0]" -o tsv)`
   - `az deployment group create --resource-group $RG --template-file labs/arm/day1/scenario03/main.json --parameters location=$LOC policyDefinitionId=$POLICY_ID policyTagName=Environment policyTagValue=Lab`
   - If deployment returns `InvalidCreatePolicyAssignmentRequest`, skip this optional step and continue to step 5.
5. Read recent activity logs for the RG:
   - `SUB_ID=$(az account show --query id -o tsv)`
   - `az monitor activity-log list --resource-group rg-az104 --max-events 20 --output table`
6. Review resource health summary (if available):
   - `az resource list --resource-group rg-az104 --output table`

## Portal Path (Alternative)
1. Open Azure Portal and go to **Resource groups** > **rg-az104**.
2. Open **Tags** on the resource group and verify expected tag values (for example Environment/Course/Day).
3. Open **Activity log** for the resource group and review recent events.
4. Open **Access control (IAM)** and confirm any policy/assignment effects if you performed the optional policy step.
5. Note down whether governance settings are directly configured at RG scope or inherited from higher scopes.

## Validation
- Tags are present on resource group.
- You can retrieve activity events for the RG.
- If policy assignment step was run, assignment is visible at RG scope.

## Complete Cleanup
- Remove policy assignment if created:
   - `SCOPE=$(az group show -n rg-az104 --query id -o tsv)`
   - `az policy assignment delete --name pa-az104-day1-tag-audit --scope $SCOPE`
- Remove day-specific tag if needed:
   - `az group update --name rg-az104 --remove tags.Day`
